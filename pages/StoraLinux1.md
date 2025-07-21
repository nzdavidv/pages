---
title: "NetGear Stora running Debian Linux Part1"
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

### Images of the console connection
<img src= "/refs/heads/main/images/IMG_0424-EDIT.jpg" alt="pic3 my very rough connecting Raspi to Stora" style="width:400px;border: 1px solid grey;">

![pic1 my very rough connecting Raspi to Stora](/pages/assets/images/IMG_0425.jpg){:width="400px"}

![pic2 my very rough connecting Raspi to Stora](/pages/assets/images/IMG_0426.jpg){:width="400px"}


## Preparing the Raspberry Pi
I used UART1, pins 27 and 28, /dev/ttyAMA1
Enabling serial port
```
$ sudo raspi-config
 Interface Options
 Serial Port
 No - to login shell over serial
 Yes - to serial port hardware enable
Then restart
```

Enabling extra UARTs
```
#vi /boot/firmware/config.txt
[all]
enable_uart=1
dtoverlay=disable-bt
dtoverlay=uart1
dtoverlay=uart2
dtoverlay=uart3
dtoverlay=uart4

Then restart
```
## Connect with minicom
```
root@raspi:~# minicom -b 115200 -o -D /dev/ttyAMA1

                                                                                                              
                                                                                                              
         __  __                      _ _                                                                      
        |  \/  | __ _ _ ____   _____| | |                                                                     
        | |\/| |/ _` | '__\ \ / / _ \ | |                                                                     
        | |  | | (_| | |   \ V /  __/ | |                                                                     
        |_|  |_|\__,_|_|    \_/ \___|_|_|                                                                     
 _   _     ____              _                                                                                
| | | |   | __ )  ___   ___ | |_ 
| | | |___|  _ \ / _ \ / _ \| __| 
| |_| |___| |_) | (_) | (_) | |_ 
 \___/    |____/ \___/ \___/ \__| 
 ** MARVELL BOARD: RD-88F6281A LE 

U-Boot 1.1.4 (Sep  4 2009 - 09:36:11) Marvell version: 3.4.14

U-Boot code: 00600000 -> 0067FFF0  BSS: -> 006CEE60

Soc: MV88F6281 Rev 3 (DDR2)
CPU running @ 1000Mhz L2 running @ 333Mhz
SysClock = 333Mhz , TClock = 200Mhz 

DRAM CAS Latency = 5 tRP = 5 tRAS = 18 tRCD=6
DRAM CS[0] base 0x00000000   size  64MB 
DRAM CS[1] base 0x04000000   size  64MB 
DRAM Total size 128MB  16bit width
Flash:  0 kB
Addresses 8M - 0M are saved for the U-Boot usage.
Mem malloc Initialization (8M - 7M): Done
NAND:256 MB
CRC in Flash: 8e6fbb12, Calculated CRC: 8e6fbb12

CPU : Marvell Feroceon (Rev 1)

Streaming disabled 
Write allocate disabled

Module 0 is RGMII
Module 1 is TDM

USB 0: host mode
PEX 0: interface detected no Link.
Net:   egiga0, egiga1 [PRIME]
Hit any key to stop autoboot:  3 
 0 
Marvell>> 

Marvell>> 

```
### Happy Days! console connected

Next blog post is <a href="StoraLinux2.html">Blog Part2</a>
