# ORB-SLAM3 
### [Link to original ORB-SLAM3's README.md](https://github.com/UZ-SLAMLab/ORB_SLAM3)

This fork incorporates the changes that I find necessary to make it easier and more straightforward to install and run ORB-SLAM3 on Ubuntu 20.04.

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
## 1.1.5 CODA
- Jump to 1.2 OpenCV for now, but if "3. Run examples" fail in the end, come back here (at own risk...)

I ran into trouble running the dataset due to an error described by someone else here: [issue 399](https://github.com/UZ-SLAMLab/ORB_SLAM3/issues/399)
First check if the path to your missing component is referenced here: “/etc/ld.so.conf" (I guess you need to follow the links)
### First try:
```
gedit /etc/ld.so.conf
#then run this to flush the cache (I guess) .
sudo ldconfig
```
### else: install homebrew:
if the library still is not working we can do this unsactioned move: [homebrew install instruction](https://www.how2shout.com/linux/how-to-install-brew-ubuntu-20-04-lts-linux/#3_Run_Homebrew_installation_script)
I've pasted the [relevant] commands here (given you have completed the steps above):
WARNING: If you google "use several package managers Linux" or simular, it comes out as a bad idea if you don't know what you're doing. So this seems like a risky move.

```
sudo apt update
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
#The above command may take a while
#Then tell the system about it's new package manager super skill:
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
#Check if all is okay
brew doctor
#then
brew install gcc
# package manger installed

```
### Have pangolin check dependencies
Now we can check dependencies of Pangolin using brew. If I understand this correctly this may override some of the installations you already have going, and migth lead to complex epic failures.
```
cd ~/Dev/Pangolin/
./scripts/install_prerequisites.sh -m brew all
#If it fails, try running it again. I had to run it twice.
cmake -B build
cmake --build build
```
### Go back to Examples and try again.


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

## 2. Build

```
# Clone the repo:
cd ~/Dev
git clone https://github.com/discoimp/ORB_SLAM3.git ORB_SLAM3

# Build
cd ORB_SLAM3
chmod +x build.sh
./build.sh
```

## 3. Run examples

[EuRoC datset](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets):

# Download
```
cd ~
mkdir -p Datasets/EuRoC
cd Datasets/EuRoC/
wget -c http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_01_easy/MH_01_easy.zip
mkdir MH01
unzip MH_01_easy.zip -d MH01/
```

# Error?
if the following examples fail with something related to Pangolin. I got "./Examples/Monocular/mono_euroc: error while loading shared libraries: libpango_windowing.so: cannot open shared object file: No such file or directory"
then go back up to the latter part of Pangolin.
# Run in mono-inertial mode
```
cd ~/Dev/ORB_SLAM3
./Examples/Monocular-Inertial/mono_inertial_euroc ./Vocabulary/ORBvoc.txt ./Examples/Monocular-Inertial/EuRoC.yaml ~/Datasets/EuRoC/MH01 ./Examples/Monocular-Inertial/EuRoC_TimeStamps/MH01.txt dataset-MH01_monoimu

```
### Live with Realsense T265:
- The param file is located inside the folder with the same name as the example that you want to run (Mono/Mono-inertial/Stereo/Stereo-Inertial). The number of parameters that you need to modify  varies accordingly.

- Run `rs-enumerate-devices -c` to obtain the intrinsic & extrinsic parameters. A good instruction with pictures can be found [here](https://github.com/shanpenghui/ORB_SLAM3_Fixed#73-set-camera-intrinsic--extrinsic-parameters).

- If necessary, calibrate the T265's IMU intrinsic with [Kalibr](https://github.com/ethz-asl/kalibr) or [imu_utils](https://github.com/shanpenghui/imu_utils). The default params seem good enough for testing.


- Run:
```
cd ~/Dev/ORB_SLAM3
./Examples/Monocular-Inertial/mono_inertial_realsense_t265 Vocabulary/ORBvoc.txt ./Examples/Monocular-Inertial/RealSense_T265.yaml 
```

## Changelog:
### 13-Aug-2022
Work with Ubuntu 20.04, no additional installation of OpenCV or C++ required:
- Update CMakeLists.txt to use OpenCV 4.2 mimimum.
- Update CMakeLists.txt to use C++14 instead of C++11.
