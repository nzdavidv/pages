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

1. Setup the physical switches
As this is a very simple lab, there are 2 unmanaged switches. 
 - switch1 has a connection to my home wifi router, a connection to the ethernet port in the ESXi laptop, and a connection to my Netgear Stora running Debian
 - switch2 has a connection to the USB-C ethernet port attached to the ESXi laptop, and a connection to the raspberry pi. You could even just use a direct connection (crossover ethernet cable) between the two.

2. Setup the ESXi laptop
 - <a href="ESXi-laptop-simpler.md">Setting up ESXi on HP Elitebook G5 laptop</a>

### Physical equipment

- Two unmanaged cheapy network switches (or just one and a crossover cable)
- An old HP Elitebook G5 laptop (to run ESXi). I got this from Facebook Marketplace for $250 NZD
- my home WiFi router
- some devices to plug into the network, a raspberry pi and an old Netgear Stora running Debian (from other blogs)
- my daily laptop (which is dual boot Ubuntu and Windows but to do the journey only need Windows) 

The lab environment consists of:
- the switch with admin and internet network. Network is the same as my home WiFi network - 192.168.30.0/24
  - Home WiFi router is plugged in
  - Debian-Stora plugged in with IP 192.168.30.117
  - The ESXi laptop is plugged in with its onboard Ethernet port. It has a DHCP reservation for IP 192.168.30.176
 
- the switch with 'ACCT' network (short for accounting I guess, I used the label from an old lab I did years ago). Network is 10.1.10.0/24
  - The ESXi laptop is plugged in with its USB-C adapter Ethernet port.
  - Raspberry Pi is plugged into this switch. Initially it has no IP address but eventually it gets one from DHCP (on the test VM) 10.1.10.10 

and then...
- Virtual Machine ACCT-VM running Debian assigned (in VMWare) to ACCT port group (which uses the USB-C adapter) - IP 10.1.10.101. 

But that's the end result. To get there wasn't quite that simple. 
- The ACCT-VM virtual machine needed changing VLANs to admin and internet network briefly to get some apt packages installed.
- GNS3 ESXi appliance needed some trial and error to get working in ESXi 8.

  
