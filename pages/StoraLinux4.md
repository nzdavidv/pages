---
NetGear Stora running Debian Linux Part4
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

I'm going to create 2 partitions on the disk, one is a small swap partition.
```
# fdisk /dev/sda

Welcome to fdisk (util-linux 2.36.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: ST31000528AS    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x000b7041

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1           1 1953125000 1953125000 931.3G fd Linux raid autodetect

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-1953525167, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1953525167, default 1953525167): 1950000000

Created a new partition 1 of type 'Linux' and of size 929.8 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 
First sector (1950000001-1953525167, default 1950001152): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1950001152-1953525167, default 1953525167): 

Created a new partition 2 of type 'Linux' and of size 1.7 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
Now format with ext4 this time (rather than ext2).. because we can. Faster better stronger.
```
# mkfs -t ext4 /dev/sda1
mke2fs 1.46.2 (28-Feb-2021)
/dev/sda1 contains a linux_raid_member file system
Proceed anyway? (y,N) y
Creating filesystem with 244140625 4k blocks and 61038592 inodes
Filesystem UUID: bf640ea7-a1be-4421-acf5-ec108d8322b3
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
	102400000, 214990848

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done

```
Label it introot, mount it
```
# e2label /dev/sda1 introot
# mkdir /media/debian
# mount /dev/sda1 /media/debian/
# chmod 755 /media/debian/

```
Setup swap partition
```
# mkswap /dev/sda2
Setting up swapspace version 1, size = 1.7 GiB (1804292096 bytes)
no label, UUID=d04b4851-b48e-4b67-8f2c-bb66f52f2a53
# swapon /dev/sda2
```

## bootstrap
Bootstrap takes quite a while.
```
# apt-get install debootstrap
# debootstrap --verbose --arch=armel stable /media/debian/ http://deb.debian.org/debian/

# cp -p /etc/inittab /media/debian/etc

```
now I copy over the entire /boot directory..

```
# cp -Rp /boot /media/debian/
```
Setup chroot
```
# mount proc /media/debian/proc -t proc
# mount sysfs /media/debian/sys -t sysfs
# mount devpts /media/debian/dev/pts -t devpts
# chroot /media/debian/ /bin/bash
root@stora:/#
```
### in chroot

Add key packages, set root passwd, add new user 'stora'
```
# apt install /boot/linux-image-6.15.2-kirkwood-tld-1_1_armel.deb
Reading package lists... Done
Building dependency tree... Done
Note, selecting 'linux-image-6.15.2-kirkwood-tld-1' instead of '/boot/linux-image-6.15.2-kirkwood-tld-1_1_armel.deb'
The following NEW packages will be installed:
  linux-image-6.15.2-kirkwood-tld-1
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B/31.7 MB of archives.
After this operation, 41.2 MB of additional disk space will be used.
Get:1 /boot/linux-image-6.15.2-kirkwood-tld-1_1_armel.deb linux-image-6.15.2-kirkwood-tld-1 armel 1 [31.7 MB]
perl: warning: Setting locale failed.                  
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
Selecting previously unselected package linux-image-6.15.2-kirkwood-tld-1.
(Reading database ... 8698 files and directories currently installed.)
Preparing to unpack .../linux-image-6.15.2-kirkwood-tld-1_1_armel.deb ...
Unpacking linux-image-6.15.2-kirkwood-tld-1 (1) ...
Setting up linux-image-6.15.2-kirkwood-tld-1 (1) ...
# apt install /boot/linux-headers-6.15.2-kirkwood-tld-1_1_armel.deb
# apt-get install  isc-dhcp-client  openssh-server  net-tools  vim nano systemd apt-utils dialog locales
# passwd root
# useradd -m -s /bin/bash stora
# passwd stora
```

Now setup apt sources, config locales, hostname, add an fstab (because without it the system kept mounting '/' as read-only).
```
# cat <<END > /etc/apt/sources.list
deb http://deb.debian.org/debian bookworm main
deb http://deb.debian.org/debian bookworm-updates main
deb http://deb.debian.org/debian-security bookworm-security main
END

# dpkg-reconfigure locales
en_NZ.UTF-8
en_US.UTF-8

# echo stora > /etc/hostname

# vi /etc/fstab
LABEL=introot / ext4 rw,discard,noatime,errors=remount-ro 0 1
/dev/sda2 swap swap defaults 0 0

root@stora:/# exit
exit
```
### back in normal OS
Prepare for reboot
```

# umount /media/debian/proc
# umount /media/debian/sys
# umount /media/debian/dev/pts
# cd /
# umount /media/debian/

# shutdown -r now
```
### change the boot to IDE
Interrupt the boot and change the boot to IDE

```

Netgear Stora> ide reset
Netgear Stora> setenv bootcmd_ide 'ide reset; ext2load ide 0:1 0x800000 /boot/uImage; ext2load ide 0:1 0x2100000 /boot/uInitrd'
Netgear Stora> setenv bootcmd 'setenv bootargs console=ttyS0,115200 root=LABEL=introot rootdelay=10 earlyprintk=serial; run bootcmd_ide; bootm 0x800000 0x2100000'
Netgear Stora> saveenv
Netgear Stora> reset

that is for disk 0. for disk 1 would be..
Netgear Stora> setenv bootcmd_ide 'ide reset; ext2load ide 1:1 0x800000 /boot/uImage; ext2load ide 1:1 0x2100000 /boot/uInitrd'
```

### after reboot
```
# dhclient etho
# apt update
# apt upgrade
# apt install network-manager

..not actually sure this did anything but I was hoping it would install a base set of packages.
# tasksel install minimal


```


## issues
none really, just /run is too small 

```
root@stora:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev             49M     0   49M   0% /dev
tmpfs            11M  660K   11M   6% /run
/dev/sda1       915G  828M  867G   1% /
tmpfs            54M     0   54M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            11M     0   11M   0% /run/user/1000
root@stora:~# free
               total        used        free      shared  buff/cache   available
Mem:          110072       55412       13680         640       55984       54660
Swap:        1762004         608     1761396


Reload daemon failed: Refusing to reload, not enough space available on /run/systemd. Currently, 10.3M are free, but a safety buffer of 16.0M is enfor
ced.
Warning: The unit file, source configuration file or drop-ins of udev.service changed on disk. Run 'systemctl daemon-reload' to reload units.
Reload daemon failed: Refusing to reload, not enough space available on /run/systemd. Currently, 10.3M are free, but a safety buffer of 16.0M is enfor
ced.
Reexecute daemon failed: Refusing to reexecute, not enough space available on /run/systemd. Currently, 10.3M are free, but a safety buffer of 16.0M is
 enforced.
Warning: The unit file, source configuration file or drop-ins of systemd-networkd.service changed on disk. Run 'systemctl daemon-reload' to reload uni
ts.

# dmesg
..had no real dramas on restart

To fix /run too small I added an entry into fstab:
root@stora:~# vi /etc/fstab 
# UNCONFIGURED FSTAB FOR BASE SYSTEM
LABEL=introot / ext4 rw,discard,noatime,errors=remount-ro 0 1
/dev/sda2 swap swap defaults 0 0
tmpfs /run tmpfs defaults,size=20M 0 0

after another restart..
stora@stora:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev             49M     0   49M   0% /dev
tmpfs            20M  664K   20M   4% /run
/dev/sda1       915G  828M  867G   1% /
tmpfs            54M     0   54M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            11M     0   11M   0% /run/user/1000

```
## still more to be refined but ultimately..
# Success!

## Post-script
I got some Plug to Plug Servo Leads from Jaycar to replace my bodgy console cables to the Raspberry Pi.
<a>https://www.jaycar.co.nz/30cm-plug-to-plug-servo-leads-pack-of-5/p/WC6040)</a>
$5.90 for a pack of 5 and they were 30cm long.

<img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/IMG_0431.jpg" alt="connecting Raspi to Stora" style="width:500px;border: 1px solid grey;">

