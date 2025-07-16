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

Wheezy is way too old but that is what I got.

If I was doing it again I would probably get the entire tarball from here: 
<a>https://forum.doozan.com/read.php?2,12096</a>
Thiss has a reference to a dropbox link..
<a>https://www.dropbox.com/scl/fi/t2zv6g1sydq019urfnsd6/Debian-5.6.7-kirkwood-tld-1-rootfs-bodhi.tar.bz2?rlkey=1uukhwhpmccvjcv8msnqbqkbd&dl=0</a>

This is an older OS but nowhere near as old as the one I started with.

## Preparing the OS

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
# tar xvf ../Debian-5.6.7-kirkwood-tld-1-rootfs-bodhi.tar.bz2

# cd /
# umount /media/david/root
```
root password: root

downloaded the tar file and this formed the basis of the kernel


