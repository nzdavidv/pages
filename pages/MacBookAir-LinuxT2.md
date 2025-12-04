---
title: "Ubuntu on Macbook Air 9,1"
date: 2025-12-04
---
# Overview & Context

MacOS seemed to be going slower and slower on my 2020 Intel MacBook Air, CPU fan going on and who knows what my computer was doing but it felt slow.
I did some research and apparently it could run Ubuntu.
This blog details the ups and downs of getting Ubuntu working on Macbook Air 9,1.

## How not to do it
So DO NOT do this.. this is what I did.
On my windows machine I downloaded Ubuntu 25 and used rufus to put it on a USB-C hard drive, and booted the Mac from it.
( hold down option, boot ).
Immediate fail as it  was set to secure boot not-allow booting anything other than macOS or Windows.

After fixing that (which I'll detail below), I re-did the boot and Ubuntu 25 started. The Keyboard and trackpad didn't work, so I used my Dell USB-C docking station and did the install.
I completely wiped out MacOS.
What a nightmare. WiFi was broken, sound was broken, I kludged together some parts but it was it was awful.
FAIL.

I tried the iso.sh script from https://wiki.t2linux.org/ on Linux a few times but for whatever reason it just didn't work. 

After the initial Ubuntu boot 'Try or install Ubuntu' it would error.

I struggled forward and eventually got WiFi kind-of working (was flakey), trackpad going, but audio was a fail (headphones wouldn't work).

So I gave up.

### To get back to MacOS 
It was a bit of a hassle to get back to Mac OS. 
I waded through https://wiki.t2linux.org/ and downloaded firmware.sh, 
then downloaded https://raw.githubusercontent.com/kholia/OSX-KVM/master/fetch-macOS-v2.py

python3 ./fetch-macOS-v2.py
8. Sequoia (15)

I wrote this to ISO, booted from it, wiped the disk and started again with MacOS.

Note that I had to setup DHCP on my router to google 8.8.8.8 8.8.4.4 to get the Mac to download Mac OS.

Well.. now that's out of the way

## Doing it the right way
follow https://wiki.t2linux.org/

### Disable secure boot

Apple's Secure Boot implementation does not allow booting anything other than macOS or Windows when it is enabled (not even shim signed GRUB). We need to disable it:

* Turn off your Mac
* Turn it on and press and hold Command-R until the black screen flashes
* Your Mac will boot in the macOS Recovery
* Select your user and enter your password
* Now, from the menu bar choose Utilities > Startup Security Utility
* Enter again the password
* Once in Startup Security Utility:
set Secure Boot to No Security
set Allow Boot Media to Allow booting from external or removable media Now you are able to boot from a Linux install ISO.
