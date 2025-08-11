---
title: "Installing Mac OS on a VM on ESXi"
date: 2025-08-11
---
# Overview
Just because you can doesn't mean you should. Technically yep it worked and I wound up with a VM running macOS Sonoma.

It's not super painfully slow but it's slow enough to be annoying. 
It certainly doesn't spark joy.. and for me it required having a physical Mac OS machine to get through the first step which is quite a hurdle.

But I guess if you needed an Apple VM for managing apple devices or some reason then yep it works.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/macosvm1.png" alt="macosvm" width="800px"></kbd>

### The 15 thousand step journey
This entire post is a refresh and short version of the really well written post <a>https://www.nakivo.com/blog/run-mac-os-on-vmware-esxi/</a>
Steps:
1. Download macOS
2. Creating a bootable macOS ISO
3. Preparing ESXi Host
4. Creating and Configuring new macOS VM
5. Installing Mac OS on the VM
6. Install VMware Tools

## 1. Download Mac OS
I bought a $200 MacBook Air 7,2 running some ancient version of the OS.
I tried my usual trick of reloading it by booting with Command-R and network reload it before doing anything else but it didn't work (couldn't find the OS to download) so I had to go another way. (The other way being allowing it on the network to download the new OS).

I downloaded the latest OS the MacBook was technically capable of running in the App store - Monterey.

Actually it wasn't quite that simple.
I used the link <a>https://apps.apple.com/us/app/macos-monterey/id1576738294?mt=12</a> and this opened in the App store, I couldn't download it but it did pop up for install under system updates.
I clicked install and waited for it to download.

I **DIDN'T** click download when done. 

## 2. Creating a bootable macOS ISO
The following in Terminal makes the ISO.  
```
Davids-MacBook-Air:~ david$ hdiutil create -o /tmp/Monterey -size 16384m -volname Monterey -layout SPUD -fs HFS+J
created: /tmp/Monterey.dmg
Davids-MacBook-Air:~ david$ hdiutil attach /tmp/Monterey.dmg -noverify -mountpoint /Volumes/Monterey
/dev/disk4          	Apple_partition_scheme         	
/dev/disk4s1        	Apple_partition_map            	
/dev/disk4s2        	Apple_HFS                      	/Volumes/Monterey
Davids-MacBook-Air:~ david$ sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/Monterey --nointeraction
Password:
Erasing disk: 0%... 10%... 20%... 30%... 100%
Making disk bootable...
Copying to disk: 0%... 10%... 20%... 30%... 40%... 50%... 60%... 100%
Install media now available at "/Volumes/Install macOS Monterey"
Davids-MacBook-Air:~ david$ hdiutil eject /Volumes/Install\ macOS\ Monterey/
"disk4" ejected.
Davids-MacBook-Air:~ david$ hdiutil convert /tmp/Monterey.dmg -format UDTO -o ~david/Documents/Monterey
Reading Driver Descriptor Map (DDM : 0)…
Reading Apple (Apple_partition_map : 1)…
Reading  (Apple_Free : 2)…
Reading disk image (Apple_HFS : 3)…
..............................................................................
Elapsed Time:  1m  4.671s
Speed: 253.3Mbytes/sec
Savings: 0.0%
created: /Users/david/Documents/Monterey.iso.cdr
Davids-MacBook-Air:~ david$ cd Documents/
Davids-MacBook-Air:Documents david$ mv Monterey.cdr Monterey.iso
Davids-MacBook-Air:Documents david$ rm /tmp/Monterey.dmg 
Davids-MacBook-Air:Documents david$ ls -al
total 33554432
drwx------+  5 david  staff          160 10 Aug 14:31 .
drwxr-xr-x+ 14 david  staff          448 10 Aug 14:25 ..
-rw-------   1 david  staff            0 10 Aug 13:58 .localized
drwxr-xr-x   3 david  staff           96 10 Aug 14:24 Install macOS Monterey.app
-rw-r--r--   1 david  staff  17179869184 10 Aug 14:30 Monterey.iso
Davids-MacBook-Air:Documents david$ 
```

## 3. Preparing ESXi Host
I was using ESXi as detailed here: <a>https://github.com/nzdavidv/pages/blob/main/pages/ESXi-laptop-simpler.md<a>
This is VMware ESXi 8.0.3 downloaded from broadcom on an HP Elitebook G5 with 16 GB RAM and 500gb NVMe drive.

In the post I originally set it up with the entire disk allocated to ESXi but now I have Windows11, Ubuntu 24, and a partition for ESXi.

**Enable SSH**
- Click Host, then click Actions > Services > Enable Secure Shell (SSH).

**Download the ESXi patch**

Originally I tried booting the MacOS VM without the patch but it didn't work - just kept cycling the Apple logo.

The patch I used was from <a>https://github.com/erickdimalanta/esxi-unlocker</a>



