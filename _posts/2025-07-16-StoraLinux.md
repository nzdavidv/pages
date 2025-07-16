---
title: "NetGear Stora running Debian Linux"
date: 2025-07-16
---
# Getting the NetGear Stora serial port functioning

I connected up the console of the Stora to my Raspberry Pi.

Following this <a>https://github.com/Dees7/openstora/blob/master/pages/Root_Access_Via_Serial_Console.md</a>

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

![pics of my very rough connecting Raspi to Stora](/assets/images/IMG_0424-EDIT.jpg)
