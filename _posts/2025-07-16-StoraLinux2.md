---
title: "NetGear Stora running Debian Linux Part2"
date: 2025-07-16
---
# Overview
There are four major pieces to this working.
1. Connecting to the console
2. The firmware - U-Boot
3. The Kernel
4. The operating system

# Section 2 - The firmware - uBoot

Maybe I could have done this running the stock OS but I didn't know what I was doing so the first step for me was to download and install an OS.
..so we're skipping to step4 - the operating system, kind-of. It's a crappy old operating system.

followed these pages 
- <a>https://forum.doozan.com/read.php?2,133873,133904,quote=1</a>
- <a>https://forum.doozan.com/read.php?3,12381</a>
- <a>https://www.cablecat.dk/sites/openstora/<a>
- <a>https://github.com/evgkirov/stora-debian-install<a>

Wheezy is way too old but that is what I got from https://github.com/evgkirov/stora-debian-install

## Preparing the USB Debian OS

```
My laptop is running Ubuntu. I plugged in a USB drive, and then unmounted it.

# umount /media/david/root1
# mkfs.ext2 /dev/sda1
mke2fs 1.47.0 (5-Feb-2023)
/dev/sda1 contains a vfat file system labelled 'USB'
Proceed anyway? (y,N) y

Label it root
# e2label /dev/sda1 root

Then transfer the contents of the tarball
# mount /dev/sda1 /media/david/root
# chmod 755 /media/david/root
# cd /media/david/root
# tar xvf ../stora-debian-rootfs_20140114.tar.gz

# cd /
# umount /media/david/root
```
root password: 111222

## Preparing to USB boot
If I could go back to old-me I'd say .. hey old me make sure you get and save a clean printenv.
Ah well.

```
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
Marvell>> printenv
...imagination...

Marvell>> setenv bootcmd_usb 'usb reset; ext2load usb 0 0x200000 /boot/uImage; ext2load usb 0 0x800000 /boot/uInitrd'
Marvell>> setenv bootcmd 'setenv bootargs console=ttyS0,115200 root=LABEL=root rootdelay=8  earlyprintk=serial; run bootcmd_usb; bootm 0x200000 0x800000'
Marvell>> saveenv
Marvell>> reset

Full output here
- <a>https://github.com/nzdavidv/pages/blob/main/assets/deb2014.txt</a>


