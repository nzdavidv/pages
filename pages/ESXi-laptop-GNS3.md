---
title: "ESXi on a HP Elitebook laptop with a Cisco 3560 switch, and virtual machine running GNS3"
date: 2025-07-25
---
# Overview
This is a multi-part series into running a Cisco router lab using GNS3 on VMware.

### Lab objective
To run a virtual Cisco router using GNS3 on a virtual machine for low cost, for no other reason really other than for the challenge itself.
My wife would say I'm 'nerding'. Sometimes I like techy projects to help keep the nerdy side of my brain happy. 

In terms of low-cost.. my Cisco switch was a freebee, the ESXi software was a free download from broadcom (although tricky site to navigate) and the laptop was fairly low cost.

### Steps overview
There are 3 main pieces to this.

1. Setup the ESXi laptop
 
 - <a href="ESXi-laptop.md">Setting up ESXi on HP Elitebook G5 laptop</a>

2. Setup the Cisco physical switch

 - <a href="Cisco-3560.md">Setting up the Cisco 3560 switch</a>

3. Setup GNS3

 - <a href="GNS3.md">Setting up GNS3 virtual machine and then configuring virtual router</a>

## Lab setup
To-Do: physical diagram and logical diagram.

### Physical equipment

- An old Cisco 3560 switch someone gave me. Any switch capable of VLANs would do.
- An old HP Elitebook G5 laptop (to run ESXi). I got this from Facebook Marketplace for $250 NZD
- my home WiFi router
- some devices to plug into the network, a raspberry pi and an old Netgear Stora running Debian (from other blogs)
- my daily laptop (which is dual boot Ubuntu and Windows but to do the journey only need Windows) 


The lab environment consists of:
- VLAN 99: admin and internet network. Network is 192.168.30.0/24
- VLAN 10: 'ACCT' network (short for accounting I guess, I used the label from an old lab I did years ago). Network is 10.1.10.0/24

The Cisco switch physical connections:
- Home WiFi router connected to port 0/1. Access port VLAN 99.
- Debian-Stora 192.168.30.117 connected to port 0/2. Access port. VLAN 99
- Raspberry Pi 10.1.10.10 connected to port 0/17. Access port. VLAN 10
- The laptop (running VMware ESXi natively) connected to port 0/18. Trunk port allowing VLANs 99, 10, 20. (haven't used 20 yet).

and then...
- The Cisco switch management IP assigned to VLAN 99 - IP 192.168.30.188. (physically using port 0/1 - access port 99).
- The ESXi management IP assigned (in VMWare) to VLAN 99 - IP 192.168.30.176. (physically using port 0/18 - trunk port). 
- Virtual Machine ACCT-VM running Debian assigned (in VMWare) to VLAN 10 - IP 10.1.10.101 (physically using port 0/18 - trunk port)

But that's the end result. To get there wasn't quite that simple. 
- The laptop for example had to be built and configured plugged directly into the home WiFi switch until the Cisco one was ready.
- The Cisco switch needed to be built using the console cable.
- The ACCT-VM virtual machine needed changing VLANs to admin and internet network briefly to get some apt packages installed.
- GNS3 ESXi appliance needed some trial and error to get working in ESXi 8.

  
