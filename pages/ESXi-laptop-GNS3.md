---
title: "ESXi on a HP Elitebook laptop with a Cisco 3560 switch, and virtual machine running GNS3"
date: 2025-07-25
---
# Overview
This is a multi-part series into running a Cisco router lab using GNS3 on VMware.

There are 3 main pieces to this.
1. Setup the Cisco physical switch

 - <a href="Cisco-3560.md">Setting up the Cisco 3560 switch</a>

2. Setup the ESXi laptop
 
 - <a href="ESXi-laptop.md">Setting up ESXi on HP Elitebook G5 laptop</a>

3. Setup GNS3

## Lab setup

The environment consists of 

- Cisco Switch connected to Ethernet router Ethernet port 0/1. VLAN 99.
- VMware ESXi physical laptop. Ethernet port 0/18 trunk port.

VLAN 99
- ESXi management IP on VLAN 99 192.168.30.176. Using port 0/18 trunk port. 
- stora 192.168.30.117. Ethernet port 0/2 access port.
- switch management IP 192.168.30.188 (assigned to VLAN) using port 0/1.

VLAN 10 (ACCT)
- Virtual Machine 10.1.10.101 (uses Ethernet port 0/18)
- Raspberry Pi 10.1.10.10 Ethernet port 0/17
  
