# Setting up a Jetson Nano (w/ Wi-Fi)

This document is meant to simplify the instructions provided in the official [NVIDIA Jetson Nano Developer Kit Guide](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#intro) along with information on how to set up Wi-Fi using the EDIMAX N150 USB Wi-Fi adapter.

There are two ways to set up a Jetson Nano:

1) With display, keyboard and mouse attached
2) In “headless mode” via connection from another computer (Recommend to deepen your understanding of the Linux CLI)

Required Items: (If missing anything, ask Varun)
1) Jetson Nano
2) Jetson power supply w/ barrel jack connector
3) MicroSD Card (Will be inside the Jetson underneath the fan)
4) Method to read & write to microSD card
5) USB to MicroUSB Data Transfer Cable


## 1. Flashing Image to MicroSD Card

1) Download the [Jetson Nano Developer Kit SD Card Image](https://developer.nvidia.com/jetson-nano-sd-card-image) (JetPack 4.6.1)
2) Install [Etcher](https://etcher.balena.io/)
3) Insert your microSD card. Select the recently downloaded image and the microSD card
4) Flash the image (will take 10-15 mins)
5) When done, insert the microSD card back into the Jetson


## 2. Initial Boot and Setup

### Display Mode

1) Jumper the J48 Power Select Header pins
2) Connect display via HDMI, USB Keyboard and Mouse
3) Plug in the Jetson and follow initial setup, selecting the default for most settings (ex. APP partition size)


### Headless Mode (Differs by OS)

1) Jumper the J48 Power Select Header pins
2) Connect computer to Jetson with the USB to MicroUSB cable

#### Linux
1) To establish a serial connection, install either the Screen program (reccomended) or [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). To install Screen: <br />

`sudo apt-get install -y screen`

2) Plug in the Jetson (will take ~1 minute to boot)
3) To confirm you are detecting the Jetson:

```shell
$ lsusb 
...
Bus 001 Device 023: ID 0955:7020 NVIDIA Corp. L4T (Linux for Tegra) running on Tegra

$ sudo dmesg | grep --color 'tty'
...
[xxxxxx.xxxxxx] cdc_acm 1-1:1.2: ttyACM0: USB ACM device
```

4) To get serial port for the Jetson: (Note that this usually will be `/dev/ttyACM0`)

```shell
ls -l /dev/ttyACM0
...
crw-rw---- 1 root dialout 166, 0 Oct  2 02:45 /dev/ttyACM0
```

5) To establish a serial connection to the Jetson, 

```shell
sudo screen /dev/ttyACM0 115200

```

6) When booting, select default settings. However, when you get to `Network Configuration`, since we will be later setting up our Wi-Fi adaptors, select `dummy0`. This will fail, allowing you to then select `Do not configure the network at this time`.
7) When finished initial setup, the Jetson will reboot, finishing initial setup.

#### Windows
1) FINISH LATER


## 3. Set up Wi-Fi Adaptors

1) Install [EDIMAX N150 drivers](https://www.edimax.com/edimax/mw/cufiles/files/download/Driver_Utility/EW-7811Un_V2/EW-7811Un_V2_Linux_Driver_1.0.1.3.zip) to a USB drive (Already have USB drives w/ the Wi-Fi drivers installed) and plug into Jetson
2) If using a display, simply copy and extract the drivers into the User directory
3) If headless, the process to extract a file from a USB Drive is more involved:

```shell
$ sudo fdisk -l  # Find the disk name: (ex. /dev/sda1)
...
/dev/sda1  *     2048 30463999 30461952 14.5G  c W95 FAT32 (LBA)

mkdir /home/$USER/usbMount  # Make a directory to mount the USB drive to
sudo mount /dev/sda1 /home/$USER/usbMount  # Mount the USB drive, change sda1 to disk game found w/ fdisk
unzip /home/$USER/usbMount/EW-7811Un_V2_Linux_Driver_1.0.1.3.zip -d /home/$USER  # Unzip the drivers to the User directory
sudo umount /home/$USER/usbMount  # Unmount the USB drive
```

4) Remove the USB Drive from the Jetson. 
5) To download the drivers:

```shell
cd /home/$USER/EW-7811Un_V2_Linux_Driver_1.0.1.3  # Enter directory of drivers
export ARCH=arm64
make
sudo make install
sudo reboot now
```

6) The Jetson will restart and you should see the USB Wi-Fi adapter flash blue as it boots up.
7) Log in. If you have a display, in the top right corner simply select the Wi-Fi network from those available in the “Wi-Fi Network” tab.
8) If you are headless, you will need to connect to Wi-Fi using the terminal:

```shell
ifconfig wlan0 # Ensure you can see an output to ensure Wi-Fi drivers are correctly installed

nmcli d  # List all of our possible network connections (Another check to ensure wlan0 is working) 
DEVICE   TYPE      STATE        CONNECTION 
wlan0    wifi      disconnected    --

nmcli r wifi on  # Turn on Wi-Fi modules

nmcli d wifi list  # Scan and list all visible WiFi networks available

sudo nmcli d wifi connect [SSID] password [PASSWORD]  # Connect to Wi-Fi with name and password

ping 8.8.8.8  # Test connection, should not return 0 as that means no connection to internet established
```

Note: You can disconnect from a Wi-Fi network w/ `sudo nmcli c down <SSID>`

## 4. Set up SSH Connection

1) Get the IP address of the jetson:

```shell
ifconfig wlan0  # Display Jetson Internet information

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet xxx.xx.x.xx  netmask 255.255.255.0  broadcast xxx.xx.x.xxx
        ...
```

The IP address is the 4 numbers seperated by three periods after inet.

2) In your host machine, connect via SSH:

```shell
ssh <Account Name>@<IP Address>  # Account name is on the right hand side of the Jetson Ubuntu CLI (accountName@Jetson)

# Example:
ssh varunJetson@123.45.6.78
```

3) Follow CLI prompts (Select Yes and enter password)
4) You are now connected to the Jetson via SSH. You can remove the USB connection and continue to interface with the Jetson


## 5. Run ROS2 Humble Docker Container

1) Download and install the [jetson-containers utilities](https://github.com/dusty-nv/jetson-containers/blob/master/docs/setup.md):
```bash
git clone https://github.com/dusty-nv/jetson-containers
bash jetson-containers/install.sh
```

2) Give yourself permission for docker commands: 

```bash
sudo usermod -aG docker $USER
```

3) Change Docker Run-Time by adding `"default-runtime": "nvidia"` to your `/etc/docker/daemon.json` configuration file.
Note that this requires you to either use vim or to install nano/gedit: `sudo <vim/nano/gedit> /etc/docker/daemon.json`

```bash
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },

    "default-runtime": "nvidia"
}
```

4) To enable these changes, restart Docker or reboot your system. 

```bash
sudo systemctl restart docker  # Restarts Docker
sudo reboot  # Restarts Jetson
```

5) Run [ROS2 Humble Docker Container][Jetson ROS Containers](https://github.com/dusty-nv/jetson-containers/tree/master/packages/ros). Note this can be run with [additional flags](https://github.com/dusty-nv/jetson-containers/blob/master/docs/run.md) depending on need: 

```bash
jetson-containers build --name=humble ros:humble-desktop  # May give an error due to python version
```

NOTE: ROS is build from source in this container. Thus, be careful not to install any addition ROS packages from apt. Instead, they must be build from source as well. There is a [helper script](https://github.com/dusty-nv/jetson-containers/blob/master/packages/ros/ros2_install.sh) provided. Likewise, there is more configurable settings (power usage, container storage, GUI etc) with instructions available in the [NVIDIA Docker Repo](https://github.com/dusty-nv/jetson-containers/blob/master/docs/setup.md).


## Jetson Tips:

1) [Jetson-stats](https://rnext.it/jetson_stats/index.html) is a powerful tool that can monitor and control your jetson. It can be downloaded with `sudo pip3 install -U jetson-stats` and run with `jtop`. (Will need to reboot after installing)


## Additional Jetson Resources:
1) [NVIDIA Jetson Nano Files](https://developer.nvidia.com/embedded/downloads#?search=Jetson%20Nano)
2) [EDIMAX Wi-Fi driver instructions](https://edimax.freshdesk.com/support/solutions/articles/14000133009-install-ew-7811un-v2-on-ubuntu-kernel-v5-4-with-official-driver)
3) [Jetson Setup Video](https://jetsonhacks.com/2019/08/21/jetson-nano-headless-setup/)
4) [Additional Wi-Fi installation instructions](https://learn.sparkfun.com/tutorials/adding-wifi-to-the-nvidia-jetson/all)
5) [Docker Container Repo](https://github.com/dusty-nv/jetson-containers/tree/master?tab=readme-ov-file)
6) [More with Docker Containers](https://jetsonhacks.com/2023/09/04/use-these-jetson-docker-containers-tutorial/)
7) [Monitoring Jetson Activity](https://jetsonhacks.com/2023/02/07/jtop-the-ultimate-tool-for-monitoring-nvidia-jetson-devices/)
