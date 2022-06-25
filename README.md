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
