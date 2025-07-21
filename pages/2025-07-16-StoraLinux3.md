---
title: "NetGear Stora running Debian Linux Part3"
date: 2025-07-16
---
# Overview
There are four major pieces to this working.
1. Connecting to the console
2. The firmware - U-Boot
3. The Kernel
4. The operating system

# Section 3 - The Kernel

At the end of the last one we had a crappy OS but an ok firmware.

We've got to jump through a few operating system upgrades before the kernel work.

You might reasonably ask 'why not just do the kernel update from Debian stretch?'.. well the answer is for me when I tried that the loading of linux-image-6.15.2-kirkwood-tld-1_1_armel.deb generated some kernel load errors and then on boot the OS decided it didn't know how to read the filesystem.
Yoinks. needed that. 

## first to Debian stretch

This updates the OS to Debian stretch

```

# vi /etc/apt/sources.list
deb http://snapshot.debian.org/archive/debian/20230101T091029Z stretch main
deb-src http://snapshot.debian.org/archive/debian/20230101T091029Z stretch main

# apt-get update
root@stora:/tmp# apt-get update
Get:1 http://snapshot.debian.org stretch Release.gpg [3,177 B]
Get:2 http://snapshot.debian.org stretch Release [118 kB]
Ign http://snapshot.debian.org stretch Release
Get:3 http://snapshot.debian.org stretch/main Translation-en [5,377 kB]                                                                              
Get:4 http://snapshot.debian.org stretch/main Sources [6,736 kB]                                                                                     
Get:5 http://snapshot.debian.org stretch/main armel Packages [6,879 kB]                                                                              
Ign http://snapshot.debian.org stretch/main Translation-en_US                                                                                        
Fetched 19.1 MB in 1min 38s (195 kB/s)
...
W: GPG error: http://snapshot.debian.org stretch Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 04EE7237B7D453EC NO_PUBKEY 648ACFD622F3D138 NO_PUBKEY 0E98404D386FA1D9 NO_PUBKEY EF0F382A1A7B6500
...

# apt-get upgrade
...
34 upgraded, 0 newly installed, 0 to remove and 81 not upgraded.
Need to get 3,805 kB of archives.
After this operation, 1,493 kB of additional disk space will be used.
Do you want to continue [Y/n]? y
WARNING: The following packages cannot be authenticated!
  base-files dash hostname sed ncurses-base mawk debian-archive-keyring libbz2-1.0 libslang2 readline-common libattr1 libacl1 libpam-runtime
  libsepol1 libuuid1 lsb-base multiarch-support sensible-utils tzdata zlib1g insserv libustr-1.0-1 adduser netbase libgpm2 libidn11 libkeyutils1
  libwrap0 ca-certificates klibc-utils libklibc libtext-wrapi18n-perl net-tools u-boot-tools
Install these packages without verification [y/N]? y
...

# apt-get dist-upgrade
# useradd -m -s /bin/bash stora
# passwd stora
stora123

# shutdown -r now
```
Full output here
- [https://github.com/nzdavidv/pages/blob/main/assets/deb-stretch.txt](https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/assets/deb-stretch.txt)

After this reboot and it gives a baseline OS to do the next step. 
```
root@stora:~# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 9 (stretch)"
NAME="Debian GNU/Linux"
VERSION_ID="9"
VERSION="9 (stretch)"
VERSION_CODENAME=stretch
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
root@stora:~# uname -r
3.10.26-stora
```

## then upgrade to buster
```
# vi /etc/apt/sources.list
deb http://snapshot.debian.org/archive/debian/20230101T091029Z buster main
deb-src http://snapshot.debian.org/archive/debian/20230101T091029Z buster main

# apt-get update
# apt-get upgrade
# apt-get dist-upgrade
# shutdown -r now
```

Full output here
- [https://github.com/nzdavidv/pages/blob/main/assets/deb-buster.txt](https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/assets/deb-buster.txt)

## then bullseye
```
# vi /etc/apt/sources.list
deb http://deb.debian.org/debian bullseye main
deb http://security.debian.org/debian-security bullseye-security main
deb http://deb.debian.org/debian bullseye-updates main

# apt-get update
N: Skipping acquire of configured file 'main/binary-armel/Packages' as repository 'http://security.debian.org/debian-security bullseye-security InRelease' doesn't support architecture 'armel'

# apt-get upgrade
# apt dist-upgrade
# shutdown -r now

root@stora:~# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

```
### take a backup at this point
Shutdown to the firmware level, and disconnect the USB drive and then attach to a computer to take a backup.
[ on my laptop ] 
```
root@david-msi:~# mkdir /media/david/root
root@david-msi:~# mount /dev/sda1 /media/david/root
root@david-msi:~# cd /media/david/root
root@david-msi:/media/david/root# tar cvf ../stora-os-bullseye.tar *
root@david-msi:~# umount /media/david/root
```
Now if it all goes bad there is a recovery point.
user stora password is stora123
user root password is still 111222

## then the kernel update

Following <a>https://forum.doozan.com/read.php?2,12096</a>

Downloaded linux-6.15.2-kirkwood-tld-1-bodhi.tar.bz2 on my laptop and bunzip'd it before transferring to the stora 
```
[ on my laptop ]
$ bunzip2 linux-6.15.2-kirkwood-tld-1-bodhi.tar.bz2
$ scp linux-6.15.2-kirkwood-tld-1-bodhi.tar root@stora:

[ on the stora ]

# apt-get install systemd

# cd /boot
# tar xvf /root/linux-6.15.2-kirkwood-tld-1-bodhi.tar
linux-image-6.15.2-kirkwood-tld-1_1_armel.deb
linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb
config-6.15.2-kirkwood-tld-1
zImage-6.15.2-kirkwood-tld-1
linux-dtb-6.15.2-kirkwood-tld-1.tar
linux-6.15.2-kirkwood-tld-1.patch

root@stora:/boot# cp -p uImage uImage.3.10.26
root@stora:/boot# cp -p uInitrd uInitrd.3.10.26

# dpkg -i ./linux-image-6.15.2-kirkwood-tld-1_1_armel.deb
Selecting previously unselected package linux-image-6.15.2-kirkwood-tld-1.
(Reading database ... 26948 files and directories currently installed.)
Preparing to unpack .../linux-image-6.15.2-kirkwood-tld-1_1_armel.deb ...
Unpacking linux-image-6.15.2-kirkwood-tld-1 (1) ...
Setting up linux-image-6.15.2-kirkwood-tld-1 (1) ...
update-initramfs: Generating /boot/initrd.img-6.15.2-kirkwood-tld-1


# dpkg -i ./linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb
Selecting previously unselected package linux-headers-6.15.2-kirkwood-tld-1.
(Reading database ... 29940 files and directories currently installed.)
Preparing to unpack .../linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb ...
Unpacking linux-headers-6.15.2-kirkwood-tld-1 (1) ...
Setting up linux-headers-6.15.2-kirkwood-tld-1 (1) ...

# tar xf linux-dtb-6.15.2-kirkwood-tld-1.tar

# cp -a zImage-6.15.2-kirkwood-tld-1  zImage.fdt
# cat dts/kirkwood-netgear_stora_ms2000.dtb  >> zImage.fdt
# mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n Linux-6.15.2-kirkwood-tld-1 -d zImage.fdt  uImage
Image Name:   Linux-6.15.2-kirkwood-tld-1
Created:      Thu Jul 17 09:07:49 2025
Image Type:   ARM Linux Kernel Image (uncompressed)
Data Size:    6027251 Bytes = 5885.99 KiB = 5.75 MiB
Load Address: 00008000
Entry Point:  00008000

# mkimage -A arm -O linux -T ramdisk -C gzip -a 0x00000000 -e 0x00000000 -n initramfs-6.15.2-kirkwood-tld-1 -d initrd.img-6.15.2-kirkwood-tld-1 uInitrd
Image Name:   initramfs-6.15.2-kirkwood-tld-1
Created:      Thu Jul 17 09:08:43 2025
Image Type:   ARM Linux RAMDisk Image (gzip compressed)
Data Size:    8939735 Bytes = 8730.21 KiB = 8.53 MiB
Load Address: 00000000
Entry Point:  00000000

root@stora:/boot# ls -altr
total 109572
-rwxr-xr-x  1 root root  1927832 Jan 13  2014 vmlinuz-3.10.26-stora
-rw-r--r--  1 root root   123716 Jan 13  2014 config-3.10.26-stora
-rw-r--r--  1 root root  1442389 Jan 13  2014 System.map-3.10.26-stora
-rw-r--r--  1 root root  1927896 Jan 13  2014 uImage.3.10.26
-rw-r--r--  1 root root  7338563 Jan 13  2014 uInitrd.3.10.26
drwxr-xr-x  2 root root     4096 May 31 23:02 dts
-rwxr-xr-x  1 root root  6015536 Jun 17 23:06 vmlinuz-6.15.2-kirkwood-tld-1
-rw-r--r--  1 root root   199045 Jun 17 23:06 config-6.15.2-kirkwood-tld-1
-rw-r--r--  1 root root  4417083 Jun 17 23:06 System.map-6.15.2-kirkwood-tld-1
-rwxr-xr-x  1 root root  6015536 Jun 18 00:42 zImage-6.15.2-kirkwood-tld-1
-rw-r--r--  1 root root  8709420 Jun 18 00:51 linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb
-rw-r--r--  1 root root 31736076 Jun 18 00:52 linux-image-6.15.2-kirkwood-tld-1_1_armel.deb
-rw-r--r--  1 root root  1525760 Jun 18 01:06 linux-dtb-6.15.2-kirkwood-tld-1.tar
-rw-r--r--  1 root root   184041 Jun 23 22:26 linux-6.15.2-kirkwood-tld-1.patch
drwxr-xr-x 21 root root     4096 Jul 17 07:50 ..
-rw-r--r--  1 root root 10449432 Jul 17 08:49 initrd.img-3.10.26-stora
-rw-r--r--  1 root root  8939735 Jul 17 09:06 initrd.img-6.15.2-kirkwood-tld-1
drwxr-xr-x  3 root root     4096 Jul 17 09:07 .
-rwxr-xr-x  1 root root  6027251 Jul 17 09:07 zImage.fdt
-rw-r--r--  1 root root  6027315 Jul 17 09:07 uImage
-rw-r--r--  1 root root  8939799 Jul 17 09:08 uInitrd

# shutdown -r now

..after restart

stora@stora:~$ su -
Password: 
root@stora:~# uname -r
6.15.2-kirkwood-tld-1
root@stora:~# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

```

## Wohoooo a decent kernel !!
<img src="https://octodex.github.com/images/welcometocat.png" height="200px" />


Next blog post is <a href="StoraLinux4.html">Blog Part4</a>
