# Noob Tutorial: Ubuntu 20.04 + ORB-SLAM3 + Ros Noetic + your own camera
[Link to original ORB-SLAM3's README.md](https://github.com/UZ-SLAMLab/ORB_SLAM3)

[Link to thien94's README.md](https://github.com/thien94/ORB_SLAM3/blob/master/README.md)

disclaimer: I started my journey in the world of Ubuntu and ROS May 2022 (i.e. a noob). These instructons are written mainly for myself as I learn. If in doubt (as you should be) follow this instead: [shanpenghui](https://github.com/shanpenghui/ORB_SLAM3_Fixed)

By following these instructions you will be able to run ORB SLAM 3 with ROS Noetic on a fresh install of Ubuntu 20.04.5 LTS, publish your camera on a ros node and run SLAM on the included datasets.

In a future update this will also cover running SLAM on your own dataset, including calibration of your camera.

## 0. Things to consider (for noobs)
### 0.1 Update your system
Make sure your system is up to date. Open a terminal [ctrl + alt + t] and run
```
sudo apt update && sudo apt upgrade
#restart if necessary
```
### 0.2 Create a new swap file (optional):
Some of the packages we are building require a certain amount of memory.
If you are running Ubuntu in a virtual environment you should allocate 12Gb of memory to Ubuntu or install Ubuntu as a dual boot to give it all your RAM.
If you only have say 8 Gb available I encourage you to follow these steps to increase your swap disk (think of it as fake RAM using HDD space) [ask Ubuntu - increase Swap](https://askubuntu.com/questions/1264568/increase-swap-in-20-04).
I've pasted the commands here:

```
swapon --show
```
Disable and delete it:
```
sudo swapoff /swapfile  
sudo rm  /swapfile
```
Don't create a swap bigger than the amount of RAM available as this [might cause issues](https://haydenjames.io/linux-performance-almost-always-add-swap-space/).
Do simple math: set swap to say 8Gb (8 * 1024 = 8192) or 16Gb (16 * 1024 = 16384).
```
#The following command will allocate 8Gb for swap:
sudo dd if=/dev/zero of=/swapfile bs=1M count=8192
```
grant root permissions and format as swap:
```
sudo chmod 600 /swapfile
sudo mkswap /swapfile
```
Be patient, my machine froze for a while during this step.
```
sudo swapon /swapfile
```
and reboot for it to take effect. To verify repeat the top command
```
reboot
```


## 1. Install Dependencies

Run the following commands in terminal
```
sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
sudo apt update
sudo apt-get install -y build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libjasper-dev libglew-dev libboost-all-dev libssl-dev
sudo apt install libeigen3-dev curl
```
## 1.1 Pangolin:
```
mkdir ~/Dev
cd ~/Dev
git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin
mkdir build && cd build
```
The source code is now on your machine and the following command will create building instructions based on dependencies listed in CMakeLists.txt (~/Dev/Pangolin/CMakeLists.txt)
```
cmake ..
```
The -j4 specifies how many jobs you would like to process simultaniously. If the build fails, try again with "-j1" before turning to google.
```
make -j4
```
If successful finish the installation by running:
```
sudo make install
```

## 1.2 OpenCV

Check the OpenCV version on your computer (required at leat 3.0 as stated in the original `README.md`):
```
python3 -c "import cv2; print(cv2.__version__)" 
```
On a freshly installed Ubuntu 20.04.4 LTS with desktop image, OpenCV 4.2.0 is included so we can skip to the next step. If a newer version is required (>= 3.0), follow the instrucions:
- [General installation instruction](https://docs.opencv.org/4.x/d0/d3d/tutorial_general_install.html). 
- If you want CUDA to be included: [How to install OpenCV 4.5.2 with CUDA 11.2 and CUDNN 8.2 in Ubuntu 20.04](https://gist.github.com/raulqf/f42c718a658cddc16f9df07ecc627be7)

For example, the main commands for OpenCV 4.4.0 (without CUDA):
```
cd ~/Dev
git clone https://github.com/opencv/opencv
git -C opencv checkout 4.4.0
gedit ~/Dev/opencv/modules/videoio/src/cap_ffmpeg_impl.hpp
```
A script just opened, now add the following lines in the top:
```
#define AV_CODEC_FLAG_GLOBAL_HEADER (1 << 22)
#define CODEC_FLAG_GLOBAL_HEADER AV_CODEC_FLAG_GLOBAL_HEADER
#define AVFMT_RAWPICTURE 0x0020
```
Then continue as usual:
```
mkdir -p ~/Dev/opencv/build
cd ~/Dev/opencv/build
cmake ..
make -j4
sudo make install
```

## 2. Install ORB SLAM 3

Clone the repo like this:
```
cd ~/Dev
git clone https://github.com/discoimp/ORB_SLAM3.git ORB_SLAM3
```
Run the shell file containing several build commands.
```
cd ORB_SLAM3
chmod +x build.sh
./build.sh
```

## 3. Run examples

[EuRoC datset](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets):

### Download datasets
```
cd ~
mkdir -p Datasets/EuRoC
cd Datasets/EuRoC/
wget -c http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_01_easy/MH_01_easy.zip
mkdir MH01
unzip MH_01_easy.zip -d MH01/
```
If you later intend to use ROS with the examples, just run this now to point it in the correct direction:
```
echo "export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:~Dev/ORB_SLAM3/Examples_old/ROS" >> ~/.bashrc
```
## Error?
if the following examples fail with something related to Pangolin. I got "./Examples/Monocular/mono_euroc: error while loading shared libraries: libpango_windowing.so: cannot open shared object file: No such file or directory"
then go to the bottom of this README to find one possible solution:
# Run in mono-inertial mode
```
cd ~/Dev/ORB_SLAM3
./Examples/Monocular-Inertial/mono_inertial_euroc ./Vocabulary/ORBvoc.txt ./Examples/Monocular-Inertial/EuRoC.yaml ~/Datasets/EuRoC/MH01 ./Examples/Monocular-Inertial/EuRoC_TimeStamps/MH01.txt dataset-MH01_monoimu

```
### Connect own event camera
For now I'm tweaking this:[rpg_dvs_ros](https://github.com/discoimp/rpg_dvs_ros)

Then learn how to calibrate your own camera:
First publish a camera node in ros:
https://automaticaddison.com/working-with-ros-and-opencv-in-ros-noetic/
Then Calibrate it:
http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration

## Pangolin (fix only if broken):
This is a bit more advanced, but not impossible.

I ran into trouble running the dataset and received the same error described by someone else here: [issue 399](https://github.com/UZ-SLAMLab/ORB_SLAM3/issues/399)
First check if the path to your missing component is referenced in this: “/etc/ld.so.conf" (follow and open the paths listed)
### First try:
```
gedit /etc/ld.so.conf
```
then run this to flush the cache (I guess)
```
sudo ldconfig
```
### else: install homebrew:
if the library still is not working we can do this unsactioned move: [homebrew install instruction](https://www.how2shout.com/linux/how-to-install-brew-ubuntu-20-04-lts-linux/#3_Run_Homebrew_installation_script)
I've pasted the [relevant] commands here (given you have completed the steps above):
WARNING: If you google "use several package managers Linux" or simular, it comes out as a bad idea if you don't know what you're doing. So this seems like a risky move, but it worked for me (so far...).

```
sudo apt update
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
The above command may take a while
Then tell the system about it's new package manager super skill:
```
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```
Check if all is okay
```
brew doctor
```
then:
```
brew install gcc
```
package manger should now be installed

### Have pangolin check dependencies
Now we can check dependencies of Pangolin using brew. If I understand this correctly this may override some of the installations you already have going, and migth lead to complex epic failures.
```
cd ~/Dev/Pangolin/
./scripts/install_prerequisites.sh -m brew all
```
If it fails, try running it again. I had to run it twice.
```
cmake -B build
cmake --build build
```
### Go back to Examples and try again.

## Install ROS Noetic on Ubuntu 20.04:
[Install Noetc on Ubuntu](http://wiki.ros.org/noetic/Installation/Ubuntu)

## Install ROS drivers for Event camera:
[Install guide Davis ++ drivers](https://github.com/discoimp/rpg_dvs_ros) -includes setting up a catkin build workspace

## Install ROS Wrapper
[Orb Slam Wrapper](https://github.com/discoimp/orb_slam3_ros_wrapper)
