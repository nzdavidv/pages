---
title: "NetGear Stora running Debian Linux Part4"
date: 2025-07-16
---
# Overview
There are four major pieces to this working.
1. Connecting to the console
2. The firmware - U-Boot
3. The Kernel
4. The operating system

# Section 4 - the operating system

So now the hard choice.. do I simply upgrade the OS (yet again) to bookworm, or, do I attempt a clean install.
The downside of the upgrade is you're left with the original thing from 2014 upgraded 37 times and I'm not 100% sure I know everything that was on the original image.

The downside of the new clean install.. it may not entirely work. it might be slower.
I decided on balance to go with the clean install, onto the IDE disk in my Stora.

## prepare the disk


switching to IDE / hard drive instead of USB

```
Netgear Stora> ide reset
Netgear Stora> setenv bootcmd_ide 'ide reset; ext2load ide 0:1 0x800000 /boot/uImage; ext2load ide 0:1 0x2100000 /boot/uInitrd'
Netgear Stora> setenv bootcmd 'setenv bootargs console=ttyS0,115200 root=LABEL=introot rootdelay=10 earlyprintk=serial; run bootcmd_ide; bootm 0x800000 0x2100000'
Netgear Stora> saveenv
Netgear Stora> reset
```
