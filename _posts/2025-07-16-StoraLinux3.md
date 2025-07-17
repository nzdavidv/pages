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

## first we need a little better OS 

```
This updates to stretch

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
- [https://github.com/nzdavidv/pages/blob/main/assets/deb2014.txt](https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/assets/deb-stretch.txt)

after this reboot and it gives a baseline OS to do the next step. 

Following <a>https://forum.doozan.com/read.php?2,12096</a>

Downloaded linux-6.15.2-kirkwood-tld-1-bodhi.tar.bz2 on my laptop and bunzip'd it before transferring to the stora 
[ on my laptop ]
$ bunzip2 linux-6.15.2-kirkwood-tld-1-bodhi.tar.bz2
$ scp linux-6.15.2-kirkwood-tld-1-bodhi.tar root@stora:

[ on the stora ]
# tar xvf linux-6.15.2-kirkwood-tld-1-bodhi.tar
# dpkg -i ./linux-image-6.15.2-kirkwood-tld-1_1_armel.deb 
Selecting previously unselected package linux-image-6.15.2-kirkwood-tld-1.
(Reading database ... 27997 files and directories currently installed.)
Preparing to unpack .../linux-image-6.15.2-kirkwood-tld-1_1_armel.deb ...
Unpacking linux-image-6.15.2-kirkwood-tld-1 (1) ...

root@stora:/home/david# dpkg -i ./linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb

tar xf ~david/linux-6.15.2-kirkwood-tld-1-bodhi.tar
tar xf  linux-dtb-6.15.2-kirkwood-tld-1.tar

# cd /boot



Next blog post is <a href="StoraLinux4.html">Blog Part4</a>
