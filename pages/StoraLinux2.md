---
NetGear Stora running Debian Linux Part2
---

# Overview
There are four major pieces to this working.
1. Connecting to the console
2. The firmware - U-Boot
3. The Kernel
4. The operating system

# Section 2 - The firmware - uBoot

Maybe I could have done this running the stock OS but first step for me was to download and install an OS.
It's an old operating system.

followed these pages 
- <a>https://forum.doozan.com/read.php?2,133873,133904,quote=1</a>
- <a>https://forum.doozan.com/read.php?3,12381</a>
- <a>https://www.cablecat.dk/sites/openstora/<a>
- <a>https://github.com/evgkirov/stora-debian-install<a>

Wheezy is in fact way too old but that is what I got from https://github.com/evgkirov/stora-debian-install

## Preparing the USB Debian OS

```
My laptop is running Ubuntu. I plugged in a USB hard drive. I have done booted with a USB stick and it worked ok as well.

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
root password that came with it: 111222

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
...use imagination...

Marvell>> setenv bootcmd_usb 'usb reset; ext2load usb 0 0x200000 /boot/uImage; ext2load usb 0 0x800000 /boot/uInitrd'
Marvell>> setenv bootcmd 'setenv bootargs console=ttyS0,115200 root=LABEL=root rootdelay=8  earlyprintk=serial; run bootcmd_usb; bootm 0x200000 0x800000'
Marvell>> saveenv
Marvell>> reset

```
Full output here
- [https://github.com/nzdavidv/pages/blob/main/assets/deb2014.txt](https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/assets/deb2014.txt)

## U-Boot firmware update

following this
- <a>https://forum.doozan.com/read.php?3,12381</a>

```
root@stora:~# tar xvf linux-tools-installation-bodhi.tar.gz 
tools/
tools/busybox
tools/e2fsck
tools/nanddump
tools/fw_printenv
tools/flash_erase
tools/nandwrite
tools/fw_env.config

root@stora:~# cp tools/fw_env.config /etc
root@stora:~# tools/fw_printenv 
arcNumber=2097
baudrate=115200
bootcmd=setenv bootargs console=ttyS0,115200 root=LABEL=root rootdelay=8  earlyprintk=serial; run bootcmd_usb; bootm 0x200000 0x800000
bootcmd_exec=run load_uimage; if run load_initrd; then if run load_dtb; then bootm $load_uimage_addr $load_initrd_addr $load_dtb_addr; else bootm $load_uimage_addr $load_initrd_addr; fi; else if run load_dtb; then bootm $load_uimage_addr - $load_dtb_addr; else bootm$load_uimage_addr; fi; fi
bootcmd_ide=ide reset; ext2load ide 0:1 0x800000 /boot/uImage; ext2load ide 0:1 0x2100000 /boot/uInitrd
bootcmd_uenv=run uenv_load; if test $uenv_loaded -eq 1; then run uenv_import; fi
bootcmd_usb=usb reset; ext2load usb 0 0x200000 /boot/uImage; ext2load usb 0 0x800000 /boot/uInitrd
bootdelay=10
bootdev=usb
console=console=ttyS0,115200
device=0:1
devices=usb ide
disks=0 1 2 3
dtb_file=/boot/dts/kirkwood-netgear_stora_ms2000
ethact=egiga0
ethaddr=52:3b:20:9c:11:51
if_netconsole=ping $serverip
ipaddr=192.168.0.231
led_error=orange blinking
led_exit=green off
led_init=green blinking
load_dtb=echo loading DTB $dtb_file ...; load $bootdev $device $load_dtb_addr $dtb_file
load_dtb_addr=0x1c00000
load_initrd=echo loading uInitrd ...; load $bootdev $device $load_initrd_addr /boot/uInitrd
load_initrd_addr=0x1100000
load_uimage=echo loading uImage ...; load $bootdev $device $load_uimage_addr /boot/uImage
load_uimage_addr=0x800000
mainlineLinux=yes
mtdids=nand0=orion_nand
mtdparts=mtdparts=orion_nand:1m(uboot),4m@1m(kernel),251m@5m(rootfs) rw
partition=nand0,2
preboot_nc=run if_netconsole start_netconsole
scan_disk=echo running scan_disk ...; scan_done=0; setenv scan_usb "usb start";  setenv scan_ide "ide reset";  setenv scan_mmc "mmc rescan"; for dev in $devices; do if test $scan_done -eq 0; then echo Scan device $dev; run scan_$dev; for disknum in $disks; do if test $scan_done -eq 0; then echo device $dev $disknum:1; if load $dev $disknum:1 $load_uimage_addr /boot/uImage 1; then scan_done=1; echo Found bootable drive on $dev $disknum; setenv device $disknum:1; setenv bootdev $dev; fi; fi; done; fi; done
serverip=192.168.0.220
set_bootargs=setenv bootargs console=ttyS0,115200 root=LABEL=rootfs rootdelay=10 $mtdparts $custom_params
start_netconsole=setenv ncip $serverip; setenv bootdelay 10; setenv stdin nc; setenv stdout nc; setenv stderr nc; version;
stderr=serial
stdin=serial
stdout=serial
uenv_addr=0x810000
uenv_import=echo importing envs ...; env import -t $uenv_addr $filesize
uenv_init_devices=setenv init_usb "usb start";  setenv init_ide "ide reset";  setenv init_mmc "mmc rescan"; for devtype in $devices; do run init_$devtype; done;
uenv_load=run uenv_init_devices; setenv uenv_loaded 0; for devtype in $devices;  do for disknum in 0; do run uenv_read_disk; done; done;
uenv_read=echo loading envs from $devtype $disknum ...; if load $devtype $disknum:1 $uenv_addr /boot/uEnv.txt; then setenv uenv_loaded 1; fi
uenv_read_disk=if test $devtype -eq mmc; then if $devtype part; then run uenv_read;  fi; else if $devtype part $disknum; then run uenv_read; fi;  fi
usb_ready_retry=15

then did this

# flash_erase /dev/mtd0 0 4
and
# nandwrite /dev/mtd0 uboot.2017.07-tld-1.netgear_ms2110.mtd0.kwb

after restart I thought it might be curtains when I saw the bad CRC message but it was ok.

U-Boot 2017.07-tld-1 (Sep 05 2017 - 00:38:05 -0700)                                                              
Netgear Stora MS2110                                                                                             
                                                                                                                 
SoC:   Kirkwood 88F6281_A1                                                                                       
DRAM:  128 MiB                                                                                                   
WARNING: Caches not enabled                                                                                      
NAND:  256 MiB                                                                                                   
*** Warning - bad CRC, using default environment                                                                 
                                                   
Then and I set the USB bbot variables again
                                                                              
Netgear Stora> setenv bootcmd_usb 'usb reset; ext2load usb 0 0x200000 /boot/uImage; ext2load usb 0 0x800000 /boot/uInitrd'
Netgear Stora> setenv bootcmd 'setenv bootargs console=ttyS0,115200 root=LABEL=root rootdelay=8  earlyprintk=serial; run bootcmd_usb; bootm 0x200000 0x800000'
Netgear Stora> saveenv   

```
### recap
So at the end of two sessions we are left with USB boot of a crappy old OS but we do have an ok firmware.

Next blog post is <a href="StoraLinux3.md">Blog Part3</a>
