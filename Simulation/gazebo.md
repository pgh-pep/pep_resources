# Gazebo & Related Topics

This document is meant to serve as an introduction to Gazebo, a key robotics simulation software. It is meant to summarize the contents found in the official [Gazebo Garden Tutorials](https://gazebosim.org/docs/garden/tutorials/), covering the ideas rather than syntax. I highly recommend anyone interested in working with Gazebo to follow the linked tutorials.


## Gazebo Versions

With recent updates, the term Gazebo can be confusing due to multiple versions. As of recent updates, the simulation software Gazebo has three main classifications. The original is Gazebo Classic (which used to be called just Gazebo). Versions of Gazebo Classic have numbers (ex. Gazebo 11). The successor to Gazebo Classic is Ignition. However, as of the beginning of 2025, Gazebo Classis has reached EOL and Ignition has been rebranded into just Gazebo. Versions of the new Gazebo are words (ex. Gazebo Garden, Gazebo Harmonic). We will be using Gazebo Garden.

Be careful getting help from online forums as the different simulation engines have slightly different uses.


## Installing Gazebo Garden

Instructions are summarized from the [official docs](https://gazebosim.org/docs/garden/install_ubuntu/#binary-installation-on-ubuntu):

If you already have ROS2 Humble installed, you might already have Gazebo 11 and Gazebo/Ignition Fortress installed. We must ensure neither are installed before installing Gazebo Garden:

```bash
sudo apt remove ign*  # Check the list of what is being removed before confirming.
sudo apt-get remove gazebo*  
```

First install necessary tools:

```bash
sudo apt-get update
sudo apt-get install lsb-release curl gnupg
```

Install Gazebo Garden:

```bash
sudo curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
sudo apt-get update
sudo apt-get install gz-garden ros-humble-ros-gzgarden ros-humble-xacro python3-sdformat13

sudo apt-get install ros-humble-ros-gzgarden
```

Reinstall Gazebo 11 (if desired):

```bash
sudo add-apt-repository ppa:openrobotics/gazebo11-gz-cli
sudo apt update
sudo apt-get install gazebo11
sudo apt install ros-humble-gazebo-ros-pkgs
```


## Introduction to Using Gazebo

Gazebo is a key robotics simulation software that works hand-in-hand with ROS. With Gazebo, we can create 3D definitions for our robot and the world we wish to test that robot in. Using ROS, we can set up a system of communication with this simulation to interact with our robot, whether this means simulating sensors to gather data (ex. GPS, IMU, camera) or controling the robot's behavior by sending commands. All of this is done through ROS communication methods, usually through subscriber-publisher relations. 

For instance, a thruster may be subscribed to a topic that describes how fast it should spin, with your own code publishing a speed and Gazebo subscribing to that speed to simulate it.

While Gazebo can be used with CLI tools, I recommend learning how to set up launch files to initiate and control simulations.


## URDF & SDFs

URDFs (Unified Robot Description Format) and SDFs (Simulation Description Format) are both methods of describing objects and environments for robot simulation.

In both, your robot will be composed of a series of links (think body parts) connected by joints. These links can have varying physical properties and can be associated with an STL or DAE file to display a custom model. Likewise, joints can be heavily customed to perform different tasks

Traditionally, URDF is supported by ROS (and has more support on online forums due to being around for longer than SDFs), is slightly simpler, and focuses only on defining robots. 

In contrast, SDF is supported by Gazebo and is slightly more complex due to its ability to define both robots and environments. However, SDF is extremely powerful, having a range of plugins that allow for easy simulation of advanced physics and sensors.

More information regarding using URDF's can be found on the [ROS2 Humble documentation](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/URDF-Main.html).

More information regarding SDFs can be found on the [Gazebo Garden documentation](https://gazebosim.org/docs/latest/building_robot/).


## Key ROS2 Packages

After creating your own URDF or SDF, the question then arises of how to communicate with Gazebo. There are a number of ROS2 packages that make this process simpler:

1) **Xacro**: Available in both CLI and Python, this tool converts Xacro files to URDFs, which can then be used to send to Gazebo.
2) **Robot State Publisher**: Broadcasts information regarding the state of the robot's links through transforms that describe each link's relation position and velocity to others.
3) **Joint State Publisher**: Publishes positions, velocities, and efforts (similar to force) of the robot's joints.
4) **Joint State Publisher GUI**: Performs the same role as the Joint State Publisher but launches a GUI to allow you to manually send messages that control joint states.
5) **Spawn Entity**: Initial publishing of your robot's description

To see the basics of implementing SDF's into a launch file, look at the [Gazebo Garden Tutorials](https://gazebosim.org/docs/garden/ros2_interop/) and the official [example project](example project) provided.


## ros_gz_bridge vs gazebo_ros

`ros_gz_bridge` and `gazebo_ros` are both packages meant to help integrate ROS with Gazebo. 

`gazebo_ros` is the older choice for this task. It allows you to use ROS topics, services, and actions to communicate with Gazebo. For instance, you can to publish sensor data from Gazebo to ROS or send commands from ROS to Gazebo.

`ros_gz_bridge` is the newer preferred choice for integrating only ROS2 with newer versions of Gazebo. It is a little more involved as it requires the creation of a `bridge.yaml` file to manually specify what information and data type you would like to bridge from ROS to Gazebo and vice versa. This allows for more flexibility and dynamic topic remapping in your projects. (Highly recommend learning how to use the bridge)

When using Gazebo Garden, it is recommended to utilize [`ros_gz_bridge`](https://gazebosim.org/docs/garden/ros2_integration/#).
