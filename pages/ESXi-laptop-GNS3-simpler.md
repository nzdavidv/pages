---
title: "ESXi on a HP Elitebook laptop, and virtual machine running GNS3, simple version"
date: 2025-07-25
---
# Overview
This is a multi-part series into running a Cisco router lab using GNS3 on VMware.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi-gns-simpler1.png" alt="esxi-gns-simpler1" width="700px"></kbd>

### Lab objective
To run a virtual Cisco router using GNS3 on a virtual machine for low cost, for no other reason really other than for the challenge itself.
My wife would say I'm 'nerding'. Sometimes I like techy projects to help keep the nerdy side of my brain happy. 

In terms of low-cost.. my Cisco switch was a freebee, the ESXi software was a free download from broadcom (although tricky site to navigate) and the laptop was fairly low cost.

### Steps overview
There are 3 main pieces to this.

1. Setup the ESXi laptop
 - <a href="ESXi-laptop.md">Setting up ESXi on HP Elitebook G5 laptop</a>
 This part is the same as the 

2. Setup the physical switches
As this is a very simple lab, there are 2 unmanaged switches. 
 - switch1 has a connection to my home wifi router, a connection to the ethernet port in the ESXi laptop, and a connection to my Netgear Stora running Debian
 - switch2 has a connection to the USB-C ethernet port attached to the ESXi laptop, and a connection to the raspberry pi

3. GNS3 simpler - modifying the setup to use USB-C ethernet and two physical NICs rather than VLAN tagging.

 - <a href="ESXi-part3.md">USB-C ethernet in ESXi & GNS3-simpler</a>


## Lab setup
To-Do: physical diagram and logical diagram.

### Physical equipment

- An old Cisco 3560 switch someone gave me. Any switch capable of VLANs would do.
- An old HP Elitebook G5 laptop (to run ESXi). I got this from Facebook Marketplace for $250 NZD
- my home WiFi router
- some devices to plug into the network, a raspberry pi and an old Netgear Stora running Debian (from other blogs)
- my daily laptop (which is dual boot Ubuntu and Windows but to do the journey only need Windows) 


The lab environment consists of:
- VLAN 99: admin and internet network. Network is the same as my home WiFi network - 192.168.30.0/24
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

  
