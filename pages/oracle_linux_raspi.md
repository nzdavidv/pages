---
title: "Installing oracle linux on Raspberry Pi"
date: 2025-08-16
---
# Overview
Downloading and installing oracle linux on raspberry pi (raspi for short).

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

--initially did the load to /dev/mmcblk0p1 and this failed.
```

## fire it up 
Turning on raspi..

no dice.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-13.png" alt="aws-www-13"  width="800px"></kbd>
```
