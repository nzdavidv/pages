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

To get back to MacOS was a bit of a hassle. 
I waded through https://wiki.t2linux.org/ and downloaded firmware.sh, 
downloaded https://raw.githubusercontent.com/kholia/OSX-KVM/master/fetch-macOS-v2.py

python3 ./fetch-macOS-v2.py
8. Sequoia (15)

I wrote this to ISO, booted from it, wiped the disk and started again with MacOS.

Note that I had to setup DHCP on my router to google 8.8.8.8 8.8.4.4 to get the Mac to download Mac OS.

Well.. now that's out of the way
