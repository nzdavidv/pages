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
This entire post is a refresh /  update (and short version) of the really well written post <a>https://www.nakivo.com/blog/run-mac-os-on-vmware-esxi/</a>
I'm fairly experienced at ESXi, MacOS, etc so the notes are brief clues for similarly experienced admins rather than screenshot by screenshot.

Steps:
1. Download macOS
2. Creating a bootable macOS ISO
3. Preparing ESXi Host
4. Creating and Configuring new macOS VM in ESXi
5. Installing Mac OS on the VM
6. Install VMware Tools
7. Upgrade to Sonoma

## 1. Download Mac OS
I bought a $200 MacBook Air 7,2 running some ancient version of the OS.
I tried my usual trick of reloading it by booting with Command-R and network reload it before doing anything else but it didn't work (couldn't find the OS to download) so I had to go another way. (The other way being allowing it on the network to download the new OS).

I downloaded the latest OS the MacBook was technically capable of running in the App store - Monterey.

I used the link <a>https://apps.apple.com/us/app/macos-monterey/id1576738294?mt=12</a> and this opened in the App store, I couldn't download it but it did pop up for install under system updates.
I clicked install and waited for it to download.

I **DIDN'T** click upgrade now or install when done. 

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

I uploaded the iso to ESXi datastore when done.

Go to Storage > Datastores and select the needed datastore. 

Click Datastore browser and then Upload in the datastore browser window.

Select Monterey.iso and click Open.

Wait a very long time. (my WiFi isn't that fast).


## 3. Preparing ESXi Host
I was using ESXi as detailed here: <a>https://github.com/nzdavidv/pages/blob/main/pages/ESXi-laptop-simpler.md<a>
This is VMware ESXi 8.0.3 downloaded from broadcom on an HP Elitebook G5 with 16 GB RAM and 500gb NVMe drive.

In the post I originally set it up with the entire disk allocated to ESXi but now I have Windows11, Ubuntu 24, and a partition for ESXi.

**Enable SSH**
- Click Host, then click Actions > Services > Enable Secure Shell (SSH).

**Download the ESXi patch**

Originally I tried booting the MacOS VM without the patch but it didn't work - just kept cycling the Apple logo.

The patch I used was from <a>https://github.com/erickdimalanta/esxi-unlocker</a>

Upload the zip file to the ESXi datastore.

Go to Storage > Datastores and select the needed datastore. 

Click Datastore browser and then Upload in the datastore browser window.

Select esxi-unlocker-master.zip and click Open.

**Install the patch**
My Datastore is called nvme-datastore
```
[root@localhost:/vmfs/volumes] cd nvme-datastore/
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5] unzip esxi-unlocker-master.zip 
Archive:  esxi-unlocker-master.zip
   creating: esxi-unlocker-301/
  inflating: esxi-unlocker-301/esxi-install.sh
  inflating: esxi-unlocker-301/esxi-smctest.sh
  inflating: esxi-unlocker-301/esxi-uninstall.sh
  inflating: esxi-unlocker-301/readme.txt
  inflating: esxi-unlocker-301/unlocker.py
  inflating: esxi-unlocker-301/unlocker.tgz
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5] chmod 755 esxi-unlocker-301/
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5] rm esxi-unlocker-master.zip 
rm: remove 'esxi-unlocker-master.zip'? y
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5] cd esxi-unlocker-301/
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5/esxi-unlocker-301] chmod 755 *
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5/esxi-unlocker-301] ./esxi-smctest.sh 
smcPresent = false
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5/esxi-unlocker-301] ./esxi-install.sh 
VMware Unlocker 3.0.1
===============================
Copyright: Dave Parsons 2011-18
Installing unlocker.tgz
Acquiring lock /var/lock/bootbank/639cd5f2-c248be29-26be-3f6e81a98ebc
INFO: Successfully claimed lock file for pid 265054
Copying unlocker.tgz to /bootbank/unlocker.tgz
Editing /bootbank/boot.cfg to add module unlocker.tgz
Success - please now restart the server!
```
Then restarted the ESXi server.

## 4. Creating and Configuring new macOS VM in ESXi
I originally installed it with 2 VCPU and 6GB RAM but later increased to 4VCPU and 8GB RAM.
- Compatibility: ESXi 7.0 U2 virtual machine
- Guest OS family: Mac OS
- Guest OS version: Apple macOS 12 (64-bit)

Originally I made the disk 40GB but if you want to upgrade to Sonoma suggest you make it 70. You can increase it (I did) by shutting down, removing any snapshots, increasting the VM disk size, then inside MacOS using the disk tool to increase the disk size ( - all pretty seamless) but it's easier just to make it large enough to begin with.

Don't forget to select the datastore ISO for the OS (Monterey.iso).

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/macosvm2.png" alt="macosvm2" width="700px"></kbd>

### Modifying the VM config
SSH in to the host again.
```
david@david-Modern-14-C12M:~/Downloads$ ssh root@192.168.30.176
(root@192.168.30.176) Password: 
The time and date of this login have been sent to the system logs.

WARNING:
   All commands run on the ESXi shell are logged and may be included in
   support bundles. Do not provide passwords directly on the command line.
   Most tools can prompt for secrets or accept them from standard input.

VMware offers powerful and supported automation tools. Please
see https://developer.vmware.com for details.

The ESXi Shell can be disabled by an administrative user. See the
vSphere Security documentation for more information.
[root@localhost:~] cd /vmfs/volumes/nvme-datastore/
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5] ls
Monterey.iso                      debtest                           macosx-2
debian-12.11.0-amd64-netinst.iso  esxi-unlocker-301                 vmkdump
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5] cd macosx-2/
[root@localhost:/vmfs/volumes/6896f53c-4c0df672-dc07-c46516f1f5b5/macosx-2] vi macosx-2.vmx

--add the line
smc.version = "0"

--Find the line:
ethernet0.virtualDev = "e1000e"

..and change e1000e to vmxnet3 for this configuration parameter:
ethernet0.virtualDev = "vmxnet3"

```

## 5. Installing Mac OS on the VM

After the macOS installer has loaded, you should see the installation wizard:

You need to select Disk Utility and partition the drive before going to install.

I chose Mac OS Extended (Journaled) and GUID I think.

Then the usual install Mac OS. The only deviation from normal is when you get to:

'Sign in with your Apple ID'. Click Set Up Later. 

I've run into problems with accepting the Terms and Conditions even on a regular macbook installing the OS and this gets around it.

## 6. Install VMware Tools
Installing VMware tools takes the experience from the worst day ever to just annoying / almost tolerable.

..you must install VMware tools. I couldn't find a newer version than darwin.iso at <a>https://packages-prod.broadcom.com/tools/frozen/darwin/</a>

Download it and upload it to the datastore same as before.

Go to Storage > Datastores and select the needed datastore. 

Click Datastore browser and then Upload in the datastore browser window.

Upload darwin.iso.

## 7. Upgrade to Sonoma

I tried updating to the latest OS but it was unusable for me so I went for Sonoma instead.

<a>https://apps.apple.com/us/app/macos-sonoma/id6450717509?mt=12</a>
