---
title: OpenNI on ROS Indigo
layout: post
tags: 
- openni 
- ros 
- ubuntu
---
This article lists down the steps to follow in order to setup OpenNI on ROS Indigo. After trying various versions of different libraries and settings, this is what ended up working for me. Note that this has only been tested in Ubuntu 14.04 64 bit with ROS Indigo and Microsoft Kinect 360. It is assumed that you already have [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) installed. 

First we have to install some peripheral Libraries:
```bash
sudo apt-get install git-core cmake freeglut3-dev pkg-config build-essential libxmu-dev libxi-dev  libusb-1.0-0-dev openjdk-6-jdk doxygen graphviz mono-complete
```

### OpenNI
Download the OpenNI package
```bash
mkdir ~/kinect
cd ~/kinect
git clone https://github.com/OpenNI/OpenNI.git
```

Compile the package using the Redist-System: 
```bash
cd OpenNI/Platform/Linux/CreateRedist/
chmod +x RedistMaker
./RedistMaker
```
And install it: 
```bash 
cd Final
tar -xjf OpenNI-Bin-Dev-Linux-x64-v1.5.7.10.tar.bz2
cd OpenNI-Bin-Dev-Linux-x64-v1.5.7.10
sudo ./install.sh
```

### SensorKinect
This part is similar to the installation of OpenNI package.

Download the package:
```bash
cd ~kinect/
git clone git://github.com/ph4m/SensorKinect.git
```
Compile package using Redist-System:
```bash
cd SensorKinect/Platform/Linux/CreateRedist/
chmod +x RedistMaker
./RedistMaker
```
Installation: 
```bash
cd Final
tar -xjf Sensor-Bin-Linux-x64-v5.1.2.1.tar.bz2
cd Sensor-Bin-Linux-x64-v5.1.2.1
sudo ./install.sh
```

### NITE

Please note that only the Versions NITE v1.5.2.21 and NITE v1.5.2.23 will work for you. Paste the nite-folder into your kinect-folder to keep track of your data:
```bash
cd ~kinect/nite/
tar -xjf NITE-Bin-Dev-Linux-x64-v1.5.2.23.tar.bz2
cd NITE-Bin-Dev-Linux-x64-v1.5.2.23
sudo ./install.sh
```

Now that everything around ROS is installed, we should reboot the system:
```bash 
sudo reboot
```

### ROS Components

After all peripheral Libraries have been installed, we can install the required ros-core-components for OpenNI-usage. But we have to delete one library before that because of some issues that will appear otherwise:

```bash
sudo apt-get remove libopenni-sensor-pointclouds0
```

Next is the installation of ROS-Kinect Driver:
```bash 
sudo apt-get install ros-indigo-openni-launch
sudo apt-get install ros-indigo-openni-camera
```

Finally we can install the openni_tracker to our usual workspace (e.g.catkin_ws):
```bash 
cd ~catkin_ws/src
git clone https://github.com/ros-drivers/openni_tracker.git 
cd ..
catkin_make
```

Now that the OpenNI tracker is installed, you’ll be able to track skeleton of a person and view it in Rviz. Open up a bunch of terminals and run the following commands.
- **Terminal 1**: `roslaunch openni_launch openni.launch`
- **Terminal 2**: `roslaunch openni_tracker openni_tracker`
- **Terminal 3**: `rosrun rviz rviz`

To view the skeleton frame in Rviz, set the base link from `map` to `openni_depth_frame` and add a TF component.
