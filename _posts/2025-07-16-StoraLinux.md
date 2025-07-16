---
title: "NetGear Stora running Debian Linux"
date: 2025-07-16
---
# Overview
There are four major pieces to this working.
1. Connecting to the console
2. The firmware - U-Boot
3. The Kernel
4. The operating system

# Section 1 - connecting to the console

I connected up the console of the Stora to my Raspberry Pi.

Following these sites: 
- <a>https://github.com/Dees7/openstora/blob/master/pages/Root_Access_Via_Serial_Console.md</a>
- <a>https://www.youtube.com/watch?v=LsNIRMNAAZ8</a>

## Connecting up physically
Pins on Stora
- 2 RXD Recieve Data line from Stora.
- 3 TXD Transmit Data line from Stora.
- 4 GND Ground.

Connecting to Raspi 5
- 27 transmit
- 28 receive
- 6 ground

Connecting the two
- Raspi 27 transmit goes to stora port 2 receive
- Raspi 28 receive goes to stora port 3 transmit
- Raspi 6 ground goes to stora 4 ground

###Images of the console connection###
![pic1 my very rough connecting Raspi to Stora](/assets/images/IMG_0424-EDIT.jpg)
![pic2 my very rough connecting Raspi to Stora](/assets/images/IMG_0425.jpg)
![pic3 my very rough connecting Raspi to Stora](/assets/images/IMG_0426.jpg)

##Preparing the Raspberry Pi
We're going to be using UART1, pins 27 and 28, /dev/ttyAMA1
$ sudo raspi-config


Connecting to the console###
I used minicom on the Raspberry Pi
