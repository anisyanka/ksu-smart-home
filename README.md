# KSU's Smart Home


## Brief description
This project and clues are used to quickly deploy own smart home assistant based on
[Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) and [Home Assistant](https://www.home-assistant.io/).


## Table of contents
 - [Deploy RPI and Home Assistant image](#deploy-rpi-and-home-assistant-image)
 - [Turn on UART on RPI and first headless login](#turn-on-uart-on-rpi-and-first-headless-login)
 - [Set Wi-Fi password and SSH connection to the board without monitors](#set-wi-fi-password-and-ssh-connection-to-the-board-without-monitors)
 - [Install and run Home Assistant (HASS)](#install-and-run-home-assistant-hass)
 - [Autostart using systemd](#autostart-using-systemd)
 - [Setup ZigBee Coordinator](#setup-zigbee-coordinator)


## Deploy RPI and Home Assistant image
Learn what you shold buy and install to deploy Home Assistant here - [Install Home Assistant Operating System](https://www.home-assistant.io/installation/raspberrypi). In the previous link you can find how to install image with the Home Assistant core already installed. You just have to flash SD card with them, but it looks like you can't run it without hdmi-display for RPI.

In my case I have no displays and I need to control the board over UART for the first booting time and then only over SSH via Wi-Fi.
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


## Set Wi-Fi password and SSH connection to the board without monitors
If you don't have access to GUI, you can setup netwotk using [command line](https://www.raspberrypi.com/documentation/computers/configuration.html#using-the-command-line) and `raspi-config`.
```
# Generate encrypted password to connect to Wi-Fi
$ wpa_passphrase <SSID> <PASSWORD>

$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
network={
	ssid="ssid"
	psk=encrypted-password
}

$ sudo reboot
```
Also, it is better to make a static IP for RPI. It is convinient to reach Raspberry Pi on the same address in your own LAN, if you often do use SSH to the board in your local LAN. Add RPI's `/etc/dhcpcd.conf` this:
```
$ sudo vim /etc/dhcpcd.conf

interface wlan0
request 192.168.1.55
```
Request the address in the DHCP DISCOVER message. There is no guarantee this is the address the DHCP server will actually give. If no address is given then the first address currently assigned to the interface is used.
It should be work. Once booting finished, but you have no ssh access to a board, find out it new IP with the help of:
```
nmap -sn 192.168.1.0/24
```
If you do really need static IP, read [this](https://www.ionos.com/digitalguide/server/configuration/provide-raspberry-pi-with-a-static-ip-address/) abd [this](https://raspberrypi.stackexchange.com/questions/37920/how-do-i-set-up-networking-wifi-static-ip-address-on-raspbian-raspberry-pi-os/74428#74428).

SSH access is might be setup by the raspi-config [tool](https://phoenixnap.com/kb/enable-ssh-raspberry-pi#ftoc-heading-4).
After this you are able to get access to the board with the help of:
```
ssh pi@192.168.1.55
```
But it still requered to type password every time you want to access.
To avoid this just [generate](https://danidudas.medium.com/how-to-connect-to-raspberry-pi-via-ssh-without-password-using-ssh-keys-3abd782688a) RSA ssh key and put public part to `~/.ssh/authorized_keys`


## Install and run Home Assistant (HASS)
Carefully do [this](https://www.home-assistant.io/installation/raspberrypi#install-home-assistant-core).
After that we can reach the server with the help of:
```
http://192.168.1.55:8123
```


## Autostart using systemd
Create `/etc/systemd/system/homeassistant.service` file in RPI or send to it with `scp`:
```
[Unit]
Description=Home Assistant
After=network-online.target

[Service]
Type=simple
User=homeassistant
WorkingDirectory=/home/homeassistant/.homeassistant
ExecStart=/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant"
RestartForceExitStatus=100
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

```
# Reload the service files to include the new service.
$ sudo systemctl daemon-reload

# Start/stop/restart your service
$ sudo systemctl start/stop/restart homeassistant

# To check the status of your service
$ sudo systemctl status homeassistant

# To obtain service logs
$ sudo journalctl -u  homeassistant.service

# To enable your service on every reboot
$ sudo systemctl enable homeassistant

# To disable your service on every reboot
$ sudo systemctl disable homeassistant
```
Read more [here](https://www.shubhamdipt.com/blog/how-to-create-a-systemd-service-in-linux/) and [here](https://community.home-assistant.io/t/autostart-using-systemd/199497).


## Setup ZigBee Coordinator
For now, I have been trying to setup only [Conbee II](https://phoscon.de/en/conbee2), therefore the next words are only about it. Before start you must update the firmware of the stick and then you are able to start to add some ZigBee devices via the **Conbee**. Otherwise butt pain will wait for you. To update the FW, use [this](https://github.com/dresden-elektronik/deconz-rest-plugin/wiki/Update-deCONZ-manually) guide.

