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

switching to IDE / hard drive instead of USB

```
Netgear Stora> ide reset
Netgear Stora> setenv bootcmd_ide 'ide reset; ext2load ide 0:1 0x800000 /boot/uImage; ext2load ide 0:1 0x2100000 /boot/uInitrd'
Netgear Stora> setenv bootcmd 'setenv bootargs $(console) root=LABEL=introot rootdelay=10 earlyprintk=serial; run bootcmd_ide; bootm 0x800000 0x2100000'
Netgear Stora> saveenv
Netgear Stora> reset
```
