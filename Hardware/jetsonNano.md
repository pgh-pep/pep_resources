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

1) Download the [Jetson Nano Developer Kit SD Card Image](https://developer.nvidia.com/jetson-nano-sd-card-image)
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

```bash
$ lsusb 
...
Bus 001 Device 023: ID 0955:7020 NVIDIA Corp. L4T (Linux for Tegra) running on Tegra

$ sudo dmesg | grep --color 'tty'
...
[xxxxxx.xxxxxx] cdc_acm 1-1:1.2: ttyACM0: USB ACM device
```

4) To get serial port for the Jetson: (Note that this usually will be `/dev/ttyACM0`)

```bash
$ ls -l /dev/ttyACM0
crw-rw---- 1 root dialout 166, 0 Oct  2 02:45 /dev/ttyACM0
```

5) To establish a serial connection to the Jetson, 
6) When booting, select default settings. However, when you get to `Network Configuration`, since we will be later setting up our Wi-Fi adaptors, select `dummy0`. This will fail, allowing you to then select `Do not configure the network at this time`.
7) When finished initial setup, the Jetson will reboot, finishing initial setup.

#### Windows
1) FINISH LATER


## 3. Set up Wi-Fi Adaptors

1) Install [EDIMAX N150 drivers](https://www.edimax.com/edimax/mw/cufiles/files/download/Driver_Utility/EW-7811Un_V2/EW-7811Un_V2_Linux_Driver_1.0.1.3.zip) to a USB drive (Already have USB drives w/ the Wi-Fi drivers installed) and plug into Jetson
2) If using a display, simply copy and extract the drivers into the User directory
3) If headless, the process to extract a file from a USB Drive is more involved:

```bash
$ sudo fdisk -l  # Find the disk name: (ex. /dev/sda1)
...
/dev/sda1  *     2048 30463999 30461952 14.5G  c W95 FAT32 (LBA)

$ mkdir /home/$USER/usbMount  # Make a directory to mount the USB drive to

$ sudo mount /dev/sda1 /home/$USER/usbMount  # Mount the USB drive, change sda1 to disk game found w/ fdisk

$ unzip /home/$USER/usbMount/EW-7811Un_V2_Linux_Driver_1.0.1.3.zip -d /home/$USER  # Unzip the drivers to the User directory

$ sudo umount /home/$USER/usbMount  # Unmount the USB drive
```

4) Remove the USB Drive from the Jetson. 
5) To download the drivers:

```bash
$ cd /home/$USER/EW-7811Un_V2_Linux_Driver_1.0.1.3  # Enter directory of drivers

$ export ARCH=arm64

$ make

$ sudo make install

$ sudo reboot now
```

6) The Jetson will restart and you should see the USB Wi-Fi adapter flash blue as it boots up.
7) Log in. If you have a display, in the top right corner simply select the Wi-Fi network from those available in the “Wi-Fi Network” tab.
8) If you are headless, you will need to connect to Wi-Fi using the terminal:

```bash
$ ifconfig wlan0 # Ensure you can see an output to ensure Wi-Fi drivers are correctly installed

$ nmcli d  # List all of our possible network connections (Another check to ensure wlan0 is working) 
DEVICE   TYPE      STATE        CONNECTION 
wlan0    wifi      disconnected    --

$ nmcli r wifi on  # Turn on Wi-Fi modules

$ nmcli d wifi list  # Scan and list all visible WiFi networks available

$ sudo nmcli d wifi connect [SSID] password [PASSWORD]  # Connect to Wi-Fi with name and password

$ ping 8.8.8.8  # Test connection, should not return 0 as that means no connection to internet established
```

## 4. Set up SSH Connection