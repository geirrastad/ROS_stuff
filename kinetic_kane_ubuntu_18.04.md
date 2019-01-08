# Installing Kinetic Kane on Ubuntu 18.04 LTS

This document will guide you through the process of installing Kinetic Kane from source to your Ubuntu 18.04 LTS. If you are new to ROS, you should probably gor for a pre-built Melodic installation.

A few notes before you embark on this mission:
   The Kinetik Desktop-full installation WILL fail, as I have not bothered to fix it (did not need the extra stuff). There are some dependencies that fail and you will manually have to build the components yourselves.
   Also, I have not done a regression test on ROS. It still may unexpectedly fail. If it do I will make a note of what went wrong here (or possibly a fix).
   

I used the [Installing from source](http://wiki.ros.org/kinetic/Installation/Source) guide, but it will fail if you try this on 18.04. It was made for 16.04. Now let's start!

## Getting ready
1) [Download and install Ubuntu](https://www.ubuntu.com/download/desktop). Make it a plain installation, and adapt to your language and default settings

2) sudo apt-get install python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential

3) sudo rosdep init (you do not need to run rosdep update after this command)

4) cd to your home directory and Clone the "rosdistro" git:
    git clone https://github.com/ros/rosdistro

5) cd rosdistro/rosdep

6) Edit the base.yaml file. Change this area
```
gazebo7:
  arch: [gazebo]
  debian:
    buster: [gazebo7]
    jessie: [gazebo7]
    stretch: [gazebo7]
  fedora: [gazebo]
  gentoo: [=sci-electronics/gazebo-7*]
  slackware: [gazebo]
  ubuntu: [gazebo7]
```
And change the "**ubuntu: [gazebo7]**" to "**ubuntu: [gazebo9]**" like this:
```
gazebo7:
  arch: [gazebo]
  debian:
    buster: [gazebo7]
    jessie: [gazebo7]
    stretch: [gazebo7]
  fedora: [gazebo]
  gentoo: [=sci-electronics/gazebo-7*]
  slackware: [gazebo]
  ubuntu: [gazebo9]
```
Then look up this part (same file):
```
  ubuntu:
    '*': [libvtk6-dev]
```
and add an line just below like this:
```
  ubuntu:
    '*': [libvtk6-dev]
    bionic: [libvtk6-dev]
```
A few lines down we also need to replace this:
```
  ubuntu:
    '*': [libvtk6-qt-dev]
```
By this:
```
  ubuntu:
    '*': [libvtk6-qt-dev]
    bionic: [libvtk6-qt-dev]
```



save the file!

7) Now edit the python.yaml and locate this area:
```
  ubuntu:
    '*': [python-pil]
    artful: [python-imaging]
```
And add a line like this:
```
  ubuntu:
    '*': [python-pil]
    bionic: [python-pil]
    artful: [python-imaging]
```
Save and exit!

8) Now go here:
    cd /etc/ros/rosdep/sources.list.d
    
    and comment out two lines in the 20-default.list:
    
```
    #yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml
    #yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml
```
    
    File content after edit:
    
```
# os-specific listings first
yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/osx-homebrew.yaml osx

# generic
# yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml
# yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml
yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/ruby.yaml
gbpdistro https://raw.githubusercontent.com/ros/rosdistro/master/releases/fuerte.yaml fuerte

# newer distributions (Groovy, Hydro, ...) must not be listed anymore, they are being fetched from the rosdistro index.yaml instead
```

9) Create a new file '10-mydistro.list' with the following content:
```
yaml file://<home_dir>/rosdistro/rosdep/base.yaml
yaml file://<home_dir>/rosdistro/rosdep/python.yaml
```
Remember to replace <home_dir> with your actual home directory!

10) run "rosdep update"

11) Now it is time to install dependencies. Run the following commands:

   sudo apt install libopencv-core3.2 libvtk6.3-qt libbullet-dev libtf2-bullet-dev python-pil libvtk6-qt-dev
    
The environment is now prepared for Kinetic source installation.

## Installing Kinetic Kane Sources

1) mkdir ~/ros_kinetic

2) Fetch the Desktop install:
```
$ rosinstall_generator desktop --rosdistro kinetic --deps --wet-only --tar > kinetic-desktop-wet.rosinstall
$ wstool init -j8 src kinetic-desktop-wet.rosinstall
```

3) Now two of these ROS packages will fail, and we will have to swap with newest git versions: rospack and geometry2
```
$ cd ~
$ git clone https://github.com/ros/rospack
$ git clone https://github.com/ros/geometry2
$ rm -rf ~/ros_kinetic/src/rospack
$ rm -rf ~/ros_kinetic/src/geometry2
$ mv rospack/ geometry2/ ros_kinetic/src/
```
We have now updated rospack and geometry2 to the latest releases
   
4) Now resolv ROS dependencies
```
$ cd ~/ros_kinetic
$ rosdep install --from-paths src --ignore-src --rosdistro kinetic -y
```