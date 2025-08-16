---
title: "Installing oracle linux on Raspberry Pi"
date: 2025-08-16
---
# Overview
Downloading and installing oracle linux on raspberry pi (raspi for short).

It doesn't work on Raspiperry Pi v5 but my older one is fine.

Following <a>https://docs.oracle.com/en/learn/oracle-linux-install-rpi</a>

## prep the memory card
Fortunately I had a spare SD I could use to give me a backout plan if this all goes pear shaped.


For some reason the download from oracle didn't go well but I could download the wget file.
```
[ on my laptop using ubuntu ]
$ chmod 755 wget.sh 
$ ./wget.sh

I unzip'd the resulting zip file

$ unzip V1050512-01.zip 
Archive:  V1050512-01.zip
  inflating: rpi-ol9.6-image-20250531.img.xz

and unmounted my micro SD and loaded the image on it

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
...
/dev/mmcblk0p1   30G   32K   30G   1% /media/david/6163-3638

$ umount /media/david/6163-3638 
$ sudo su -
# xzcat rpi-ol9.6-image-20250531.img.xz > /dev/mmcblk0

```

## fire it up 
Turning on raspi..

ok so SUPER ANNOYING that you can't ssh in as root to begin with. I had to connect a monitor and keyboard (fortunately I had an HDMI to mini-HDMI or whatever it is called).
After that yep could login with root and oracle and add my user account.

Now I can SSH in.
```
david@david-Modern-14-C12M:~/Downloads$ ssh david@raspidev.nzvink.com
The authenticity of host 'raspidev.nzvink.com (192.168.10.100)' can't be established.
ED25519 key fingerprint is SHA256:hZ6KvEVLNiU7u5SCcep71hhuJeqxtt/kU+Fz3RJ+1Uo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'raspidev.nzvink.com' (ED25519) to the list of known hosts.
david@raspidev.nzvink.com's password: 
Last login: Sat Aug 16 03:15:33 2025 from 192.168.10.101
```
### resize the root filesystem
```
[david@rpi ~]$ su -
Password: 
Last login: Sat Aug 16 03:15:38 UTC 2025 on pts/0
Last failed login: Sat Aug 16 03:21:26 UTC 2025 on pts/1
[root@rpi ~]# mount|grep root
/dev/mmcblk0p3 on / type btrfs (rw,relatime,seclabel,ssd,discard=async,space_cache=v2,subvolid=257,subvol=/root)
[root@rpi ~]# growpart /dev/mmcblk0 3
CHANGED: partition=3 start=788480 old: size=6815744 end=7604223 new: size=61545439 end=62333918
[root@rpi ~]# btrfs filesystem resize max /
Resize device id 1 (/dev/mmcblk0p3) from 3.25GiB to max
# yum update

```
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-13.png" alt="aws-www-13"  width="800px"></kbd>
```
