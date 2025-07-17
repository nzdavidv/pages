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

Following <a>https://forum.doozan.com/read.php?2,12096</a>

Downloaded linux-6.15.2-kirkwood-tld-1-bodhi.tar.bz2 on my laptop and bunzip'd it before transferring to the stora 
```
[ on my laptop ]
$ bunzip2 linux-6.15.2-kirkwood-tld-1-bodhi.tar.bz2
$ scp linux-6.15.2-kirkwood-tld-1-bodhi.tar root@stora:

[ on the stora ]


# cd /boot
# tar xvf /root/linux-6.15.2-kirkwood-tld-1-bodhi.tar
linux-image-6.15.2-kirkwood-tld-1_1_armel.deb
linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb
config-6.15.2-kirkwood-tld-1
zImage-6.15.2-kirkwood-tld-1
linux-dtb-6.15.2-kirkwood-tld-1.tar
linux-6.15.2-kirkwood-tld-1.patch

# dpkg -i ./linux-image-6.15.2-kirkwood-tld-1_1_armel.deb
Selecting previously unselected package linux-image-6.15.2-kirkwood-tld-1.
(Reading database ... 26948 files and directories currently installed.)
Preparing to unpack .../linux-image-6.15.2-kirkwood-tld-1_1_armel.deb ...
Unpacking linux-image-6.15.2-kirkwood-tld-1 (1) ...
Setting up linux-image-6.15.2-kirkwood-tld-1 (1) ...
update-initramfs: Generating /boot/initrd.img-6.15.2-kirkwood-tld-1
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/fs/btrfs/btrfs.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/fs/ext2/ext2.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/cdrom/cdrom.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/fs/isofs/isofs.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/fs/nls/nls_ucs2_utils.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/fs/jfs/jfs.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/lib/crc-itu-t.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/fs/udf/udf.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/fs/xfs/xfs.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/scsi/scsi_transport_fc.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/message/fusion/mptbase.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/message/fusion/mptscsih.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/message/fusion/mptfc.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/scsi/scsi_transport_sas.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/message/fusion/mptsas.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/scsi/scsi_transport_spi.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/message/fusion/mptspi.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/firewire/firewire-core.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/firewire/firewire-ohci.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/drivers/firewire/firewire-sbp2.ko.xz not found.
modinfo: ERROR: Module /lib/modules/6.15.2-kirkwood-tld-1/kernel/crypto/blake2b_generic.ko.xz not found.

# dpkg -i ./linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb
Selecting previously unselected package linux-headers-6.15.2-kirkwood-tld-1.
(Reading database ... 29940 files and directories currently installed.)
Preparing to unpack .../linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb ...
Unpacking linux-headers-6.15.2-kirkwood-tld-1 (1) ...
Setting up linux-headers-6.15.2-kirkwood-tld-1 (1) ...

# tar xf linux-dtb-6.15.2-kirkwood-tld-1.tar

# mv uImage uImage.3.10.26
# mv uInitrd uInitrd.3.10.26
# cp -a zImage-6.15.2-kirkwood-tld-1  zImage.fdt
# cat dts/kirkwood-netgear_stora_ms2000.dtb  >> zImage.fdt
# mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n Linux-6.15.2-kirkwood-tld-1 -d zImage.fdt  uImage
Image Name:   Linux-6.15.2-kirkwood-tld-1
Created:      Thu Jul 17 06:39:48 2025
Image Type:   ARM Linux Kernel Image (uncompressed)
Data Size:    6027251 Bytes = 5885.99 kB = 5.75 MB
Load Address: 00008000
Entry Point:  00008000

# mkimage -A arm -O linux -T ramdisk -C gzip -a 0x00000000 -e 0x00000000 -n initramfs-6.15.2-kirkwood-tld-1 -d initrd.img-6.15.2-kirkwood-tld-1 uInitrd
Image Name:   initramfs-6.15.2-kirkwood-tld-1
Created:      Thu Jul 17 06:40:18 2025
Image Type:   ARM Linux RAMDisk Image (gzip compressed)
Data Size:    3328726 Bytes = 3250.71 kB = 3.17 MB
Load Address: 00000000
Entry Point:  00000000

# shutdown -r now
```



Next blog post is <a href="StoraLinux4.html">Blog Part4</a>
