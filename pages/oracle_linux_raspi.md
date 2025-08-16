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
# yum update

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-13.png" alt="aws-www-13"  width="800px"></kbd>
```
