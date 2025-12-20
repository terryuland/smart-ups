# Smart UPS
Using a Raspberry Pi to add smarts to a standard UPS with a USB interface

<img alt="UPS and Raspberry Pi" src="UPSRPI.JPEG">

## Overview
A [Rasbperry Pi](https://www.raspberrypi.com/) can be used to give remote management capabilities to a standard consumer-grade UPS that only has a USB port. The RPi runs a software suite called [Network UPS Tools (NUT)](https://networkupstools.org/) allowing for remote monitoring and commands via wifi. This can be combined with a platform like [Home Assistant](https://home-assistant.io) and a smart plug to provide a way to shut down the UPS along with connected devices completely and then bring it back online when necessary. 

## Rasberry Pi Configuration
Raspberry Pi Imager can be used to write the operating system to the SD card. 
1. Select Raspberry Pi Zero 2 W for the Device
2. Select Raspberry Pi OS Lite (64-bit) [NOTE: You may need to select "Other" for this to appear]
3. Set up your username, password, and Wifi info under Customisation
4. Enable SSH (also under Customisation)
5. Write the SD card, plug it in to your Pi, and plug your Pi into power and to the UPS
6. You may need to use an IP scanner or check your DHCP logs to find the IP of the Pi
7. Connect to the Pi using [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) (or any other SSH client) and log in

## Unattended Upgrades
This will install the [unattended-upgrades](https://wiki.debian.org/PeriodicUpdates) package and configure it
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y unattended-upgrades
echo unattended-upgrades unattended-upgrades/enable_auto_updates boolean true | sudo debconf-set-selections
sudo dpkg-reconfigure -f noninteractive unattended-upgrades
```

## NUT Configuration
Install NUT with `sudo apt-get install nut-server`
Edit the following files using your favorite text editor:
### /etc/nut/nut.conf
Change `MODE` to `netserver`
### /etc/nut/ups.conf
Add a block for the UPS unit connected. AFAIK all USB units use the `usbhid-ups` driver. If you only have one UPS, you can use `auto` for the port.
```
[name-of-ups]
  driver = usbhid-ups
  port = auto
```
### /etc/nut/upsd.conf
Add a line to make it listen for remote connections.
`LISTEN 0.0.0.0 3493`
### /etc/nut/upsd.users
Add a block for your admin user and password that will have full control.
```
[admin]
  password = somereallysecretpassword
  actions = SET
  instcmds = ALL
```
Restart NUT after editing the files with `sudo /etc/init.d/nut-server restart`
You can test the installation with the following commands:

`upsc -l` will list the UPS units on the local machine

`upsc upsname@localhost` will print the data from the ups called `upsname`

## Home Assistant Configuration
If the NUT server is on the same network segment as your Home Assistant server, it should be auto-discovered and show up under Integrations. If it does not, you'll need to [manually add the Network UPS Tools (NUT) integration](https://www.home-assistant.io/integrations/nut/#configuration)

Once the integration is installed and configured you should be able to see everything the UPS reports. You can also write automations that turn off the load immediately, turn it off and then turn it back on when power returns, turn on or off the buzzer, and so forth. You can see all the commands available on your UPS by running `upscmd -l upsname` from the RPi console.

You can also select from a drop-down list from within the Automation Editor. Click on Add action, then Device, type in "nut" to search for your UPS, and select it. The Action menu with the drop down appears just underneath the Device.

## YouTube Video
[![Build a Smart UPS with a Raspberry Pi](http://img.youtube.com/vi/hevZpf9laTU/0.jpg)](http://www.youtube.com/watch?v=hevZpf9laTU)

## Bill of Materials

|	Description	                                                      | Link                                  | Cost    |
|-------------------------------------------------------------------|---------------------------------------|---------|
| Raspberry Pi Zero 2 W                                             | https://www.amazon.com/dp/B09LH5SBPS  |  $29.99 |
| Compact Raspberry Pi Zero Case                                    | https://www.amazon.com/dp/B0CXWFX783  |  $4.99  |
| Micro USB to USB B Cable (2 Pack)                                 | https://www.amazon.com/dp/B099N1PWW6  |  $9.99  |
| USB A to Micro USB Cable (5 Pack)                                 | https://www.amazon.com/dp/B095JZSHXQ  |  $6.99  |
| 32GB Micro SD Card                                                | https://www.amazon.com/dp/B0C1Y87VT3  |  $9.99  |
| Shelly Smart Plug                                                 | https://www.amazon.com/dp/B096W3ZZDD  |  $23.00 |
| SD Card Reader                                                    | https://www.amazon.com/dp/B081VHSB2V  |  $9.99  |



## Links and Resources

[Raspberry Pi Imager](https://www.raspberrypi.com/software/)

[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

[Network UPS Tools](https://networkupstools.org/)

[Home Assistant](https://home-assistant.io)
