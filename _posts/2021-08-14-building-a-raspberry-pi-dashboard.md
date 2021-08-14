---
layout: default
title: "Building a Raspberry Pi dashboard display"
description: "How to make any television into an awesome cycling dashboard"
# image: "https://s3-eu-west-1.amazonaws.com/cityringen/comparisons/2018-2019/2018-2019-share.png"
---

# Building a Raspberry Pi dashboard display

Imagine you have a spare television or monitor at work, not being used for anything, and you want to use it to display some websites in tabs, such as information dashboards. You want it to start in fullscreen mode and automatically loop through the tabs. Ideally you want this to start up just by turning the television on, and you want it to work independently without the need for a keyboard or mouse attached. Anyone should be able to configure which tabs are displayed by updating a document in GitHub.

Grab your Raspberry Pi, because we're about to build a dashboard display!

## You will need

* A spare television
* A Raspberry Pi 4 (other models will work but generation 4 is silent, has built-in WiFi and has up to 4K HDMI output)
* A micro-HDMI to HDMI cable
* A USB to USB C to power the Raspberry Pi from USB output on the television
* A keyboard and mouse to set things up initially
* Possibly: An SD card reader to write a new operating system to SD card

## What this tutorial is not

This tutorial assumes you already have some dashbard websites on the internet that you want to display, maybe from Google Analytics, Kibana, Grafana, etc. This tutorial does not cover the creation of such dashboards.

## Start with Raspberry Pi OS

My recommendation is you head to [raspberrypi.org](https://www.raspberrypi.org) and download the latest Raspberry Pi OS imager. If you've just bought a new Raspberry Pi, it's likely it already has an OS installed, so you can skip this step.

Connect your SD card reader to your computer and go through the imaging process to write Raspberry Pi OS (32-bit) to the SD card.

## Initial Raspberry Pi setup

Connect your Raspberry Pi to the television HDMI and USB port, and it should start to boot up. At this point you'll want to use a keyboard and mouse connected to the Raspberry Pi to set up the initial configuration.

Follow the tutorial to set location, set a password, and connect to a WiFi network. Now reboot.

## Set up Chromium browser

Go to the Raspberry Pi menu -> Internet -> Chromium browser

Search for ant install TabCarousel plugin

Go to the plugin preferences and set how often you want it to cycle between tabs, and set it to start automatically.

## Advanced Raspberry Pi setup

Go to the Raspberry Pi menu -> Settings -> Raspberry Pi Configuration

Under System, set a hosthame (I set mine to `piboard`).

Make sure it is set to boot to desktop, and auto login as user 'pi'.

Under Interfaces, enable SSH.

Reboot again.

## Check SSH access

Of course it is possible to follow this entire tutorial with a keyboard and mouse attached directly to the Raspberry Pi, but that's not always the most practical solution. So you can go to another computer connected to the same WiFi network, and ssh in.

    ssh pi@piboard.local

Accept the connection by typing `yes` and type the password that you configured for the Raspberry Pi, and you will be in!

## Disable low voltage warnings

If you see a lightning bolt symbol in the corner of the screen, it's a warning that your television USB output is not supplying the ideal amount of power to the Raspberry Pi. There is a small risk that the Raspberry Pi could start to behave strangely or corrupt the SD card. If this concerns you, get a proper plug adapter and power your Raspberry Pi from a standard power socket.

In my experience the risk is very small, and in the worst case scenario I would just reinstall Raspberry Pi OS and follow this tutorial again. I like the Raspberry Pi being powered by the television because it can be turned on and off just with a TV remote. So i choose to disable the warnings.

If you want to do this, it's time to start editing a file on the Raspberry Pi. From your SSH terminal:

    sudo nano /boot/config.txt

Add the following lines at the end of the file:

    # Disable under-voltage warning
    avoid_warnings=1

Save the file and reboot the Raspberry Pi. You can do this remotely:

    sudo reboot

This will deal with the lighthing bolt symbol. You may find you still see a grey warning box. If this is the case, you can disable this too:

    sudo apt-get remove lxplug-ptbatt

## Install unclutter

Unclutter is a little program to hide the mouse pointer, to prevent it getting in the wy when you don't need it. We're going to use this to keep our dashboard lovely and tidy.

    sudo apt-get install unclutter

## Create a links file

Add a text file to a public GitHub repo with the links you want to open:

![List of dashboard links](/images/piboard/list-of-dashboard-links.png)

Of course, you could put this file into a private repo if security is a concern, but you would have to work a bit harder with the script that's going to read it to open the tabs.

## Write a script to launch the dashboard

Create a dashboard.sh file

    sudo nano /home/pi/dashboard.sh

Save this script to the file:

    #!/bin/bash

    # config options
    export ORG=drdk
    export REPO=dashboards
    export BRANCH=main
    export DASHBOARD=valhal

    # don't let this display sleep
    export DISPLAY=:0.0
    xset s noblank
    xset s off
    xset -dpms

    # hide mouse pointer
    unclutter -idle 0.5 -root &

    # trick Chromium into believing it shut down successfully last time
    sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' /home/pi/.config/chromium/Default/Preferences
    sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' /home/pi/.config/chromium/Default/Preferences

    # Launch Chromium 
    /usr/bin/chromium-browser \
      --start-fullscreen \
      --no-first-run \
      --noerrdialogs \
      --disable-infobars \
      $(curl -s "https://raw.githubusercontent.com/${ORG}/${REPO}/${BRANCH}/${DASHBOARD}.txt") &

Some notes about this script:

* The config options specify

Make it executable

sudo chmod a+x /home/pi/dashboard.sh



Try it out

/home/pi/dashboard.sh



Make a service to start up the dashboard

sudo nano /etc/systemd/system/dashboard.service


[Unit]
Description=Chromium Dashboard
After=network.target
After=systemd-user-sessions.service
After=network-online.target
After=graphical.target
Requires=graphical.target
Requires=network-online.target

[Service]
Environment=DISPLAY=:0.0
Environment=XAUTHORITY=/home/pi/.Xauthority
Type=forking
ExecStart=/home/pi/dashboard.sh
ExecStartPre=/bin/sh -c 'until ping -c1 raw.githubusercontent.com; do sleep 1; done;'
Restart=on-abort
User=pi
Group=pi

[Install]
WantedBy=graphical.target



Enable the service to run at boot

sudo systemctl enable dashboard


Start the service

sudo systemctl start dashboard


Stop the service

sudo systemctl stop dashboard



In case you have changed the service definition

sudo systemctl daemon-reload



Debug the service

sudo journalctl -u dashboard







Enable Emoji in browser

sudo apt-get install fonts-noto-color-emoji

sudo fc-cache -f -v




https://jonathanmh.com/raspberry-pi-4-kiosk-wall-display-dashboard/
