# Setting up ROS2 Humble

This document is meant to simplify the [installation instructions](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html) for Ubuntu 22.04 and WSL and provide an introduction to `rosdep`.

Make sure to have [VSCode](https://code.visualstudio.com/download) installed.

## ROS2 Installation

### 1. Installing Ubuntu 22.04 w/ WSL2 (Skip if using Ubuntu)

Make sure Windows is up to date.

NOTE: WSL install instructions vary depending on Windows version. For help on any errors that may occur:

- <https://learn.microsoft.com/en-us/windows/wsl/install-manual>
- <https://learn.microsoft.com/en-us/windows/wsl/troubleshooting>

To install WSL2, in a PowerShell or Windows Command Prompt run:

```powershell
wsl --install -d Ubuntu-22.04
```

You will be promted to enter a username and password followed by a successful installation message.
(Your password will not be shown when typing for security reasons)

To confirm a successful installation of WSL and Ubuntu, you can ensure WSL is set to version 2 andlist your currently installed distros with in powershell:

```powershell
wsl --set-default-version 2
wsl --list -v
```

### 2. Installing ROS2 Humble

Open the bash terminal of your newly installed distro by opening the Ubuntu app.

Update and upgrade packages:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Add the ROS 2 apt repository to your system:

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt-get update && sudo apt-get upgrade -y
```

Install ROS2 Humble:

```bash
sudo apt install ros-humble-desktop -y
```

ROS must be sourced every time the terminal is launched. The following will  add ROS to the `~/.bashrc` file to be automatically sourced and add/source colcon autocompletion.

```bash
sudo apt-get install python3-colcon-common-extensions -y
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash" >> ~/.bashrc
source ~/.bashrc
```

```bash
sudo apt-get install python3-rosdep2 python3-colcon-common-extensions libsdl1.2-dev -y
sudo rosdep init
rosdep update
```

### 3. Installing System Dependencies

Install other Ubuntu dependencies:

```bash
sudo apt-get install bash-completion gedit python3-pip python-is-python3 -y
```

Head over to [`gazebo.md`](../Simulation/gazebo.md) on instructions to install Gazebo Garden. 


## Managing ROS Dependencies w/ rosdep

[`rosdep`](https://docs.ros.org/en/humble/Tutorials/Intermediate/Rosdep.html#what-is-rosdep) is a key tool thak makes installing and managing ROS dependencies much easier (ex. `joint_state_publisher`, `common_interfaces`). You generally want to ensure you have all dependencies before building a package.

In simple terms, `rosdep` goes through your packages and checks each packages `package.xml` file. These files are parsed to find the contents inside of `<build_depend>`, `<exec_depend>`, or `<depend>`, known as package identifiers call keys. These keys are then cross-referenced against a [central ROS repository](https://github.com/ros/rosdistro/blob/master/humble/distribution.yaml) to find the location of a package and to install it.

Note that `rosdep` can manage non-ROS packages, in particular there exist keys the keys for [`apt` system dependencies](https://github.com/ros/rosdistro/blob/master/rosdep/base.yaml) and [Python system dependencies](https://github.com/ros/rosdistro/blob/master/rosdep/python.yaml).

However, at the moment, all you have to focus on is installing those dependencies rather than adding new ones to the list. If this is your first time using `rosdep`, it must be installed and initialized:

```bash
apt-get install python3-rosdep  #  Install rosdep
sudo rosdep init                #  Initialize rosdep
rosdep update                   #  Update local rosdep list of keys
```

Then, in your workspace containing ROS packages, install ROS dependencies:

```bash
rosdep install --from-paths src -y --ignore-src
```
