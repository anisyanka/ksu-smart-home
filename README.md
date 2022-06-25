# KSU's Smart Home


## Brief description
This project and clues are used to quickly deploy own smart home assistant based on
[Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) and [Home Assistant](https://www.home-assistant.io/).


## Deploy RPI and Home Assistant image
Learn what you shold buy and install to deploy Home Assistant here - [Install Home Assistant Operating System](https://www.home-assistant.io/installation/raspberrypi). In the previous link you can find how to install image with the Home Assistant core already installed. You just have to flash SD card with them, but it lookS like you can't run it without hdmi-display for RPI.

In my case I have no it and I need to control the board over UART for the first booting time and then only over SSH via Wi-Fi.
Therefore in my case I am going to install the [Raspberry Pi OS](https://www.raspberrypi.com/software/) (image without preinstalled apps) and the Home Assistant core by [hand](https://www.home-assistant.io/installation/raspberrypi#install-home-assistant-core).

Install prerequisites libs on your Linux machine and flash a SD card:
```sh
$ sudo apt-get install build-essential qt5-default libqt5quickcontrols2-5 libqt5multimedia5 libqt5webengine5 libqt5quick5 libqt5qml5

# It is for ubuntu 16.04.
# For older versions act like in the link above
$ snap install rpi-imager

# Insert SD card and install Raspberry Pi OS 64 bit
$ rpi-imager
```

After image have been flashed to SD card, you can seek for `config.txt` and cmdline.txt files on it. It is files for bootloader to config the Linux kernel and command line string.


## Turn on UART on RPI and first headless login
Attach USB-UART converter to laptop and find out what device has appeared using `dmesg`. For instance, `/dev/ttuUSB0`. And then:
 - To enable UART output add the option `enable_uart=1` at the end of `/boot/config.txt`.
 - Check what the baudrate for UART is in `cmdline.txt`.

```sh
 $ sudo apt-get install tio putty
 $ sudo chmod 666 /dev/ttyUSB0
 $ tio /dev/ttyUSB0 -b 115200

 # Or with GUI
 $ putty
```
 Wait for the board boot will be finished and log in with these defaults:
 ```
 Username: pi
 Password: raspberry
 ```
**Lessons learned:**
 - Remove `console=tty1` phrase from `cmdline.txt` if you want to use consol over UART. `cmdline` should look like: `console=serial0,115200 root=PARTUUID=fc181407-02 rootfstype=ext4 fsck.repair=yes rootwait`

 - If the default password is not working, add `init=/bin/sh` to the cmdline and than remount fs as RW and change password for `pi` user. See more [this](https://windowsreport.com/raspberry-pi-password-not-working/).
 ```
# mount -o remount, rw / 
# passwd pi
# sync
# exec /sbin/init

Then remove `init=/bin/sh` from cmdline.
 ```

If you want to see kernel messages during boot process, remove the word `quiet` in the `cmdline.txt` file to allow display of boot activity.
