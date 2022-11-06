# Noob Tutorial: Ubuntu 20.04 + ORB-SLAM3 + Ros Noetic + your own camera
### [Link to original ORB-SLAM3's README.md](https://github.com/UZ-SLAMLab/ORB_SLAM3)
### [Link to thien94's README.md](https://github.com/thien94/ORB_SLAM3/blob/master/README.md)

By following these instructions you will be able to install ORB SLAM 3 with ROS Noetic on a fresh install of Ubuntu 20.04.5 LTS, publish your camera on a ros node and run SLAM on the included datasets.

In a future update this will also cover running SLAM on your own dataset, including calibration of camera.

## 0. Things to consider (for noobs)
Some of the packages we are building require a certain amount of memory.
If you are running Ubuntu in a virtual environment you should allocate 12Gb of memory to Ubuntu or install Ubuntu as a dual boot to give it all your RAM.
If you only have say 8 Gb available I encourage you to follow these steps to increase your swap disk (think of it as fake RAM using HDD space) [ask Ubuntu - increase Swap](https://askubuntu.com/questions/1264568/increase-swap-in-20-04).
I've pasted the commands here:

### 0.1 Create a new swap file:
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


## 1. Install

### Dependencies
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
cmake ..
make -j4
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

For example, the main commands for OpenCV 4.4.0 without CUDA and other bells and whistles:
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

Clone the repo
```
cd ~/Dev
git clone https://github.com/discoimp/ORB_SLAM3.git ORB_SLAM3
```
Run the shell file containing the build commands. If you later need to build only 
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

## Error?
if the following examples fail with something related to Pangolin. I got "./Examples/Monocular/mono_euroc: error while loading shared libraries: libpango_windowing.so: cannot open shared object file: No such file or directory"
then go back up to the latter part of Pangolin.
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

## Changelog:
### 13-Aug-2022
Work with Ubuntu 20.04, no additional installation of OpenCV or C++ required:
- Update CMakeLists.txt to use OpenCV 4.2 mimimum.
- Update CMakeLists.txt to use C++14 instead of C++11.
