---
title: "ESXi using USB-C ethernet"
date: 2025-07-28
---
# Overview
Continued fun in the lab but instead of using VLAN tagging using two seperate NICs.
One of them is USB-C ethernet interface.

### Recap where things are at
- Cisco switch fully configured for the lab. management IP 192.168.30.188
- ESXi laptop is plugged into the trunk port 0/18 and admin network configured to VLAN 99. Management IP 192.168.30.176
- Test VM is on ACCT VLAN 10 with a fixed IP 10.1.10.101, running a DHCP server
- Raspberry pi is on the VLAN 10 plugged into switchport 0/17 VLAN 10. It now has an IP 10.1.10.10
- Debian stora is plugged into port 0/2 on VLAN 99 IP 192.168.30.117
- GNS3 VM loaded successfully in ESXi and configured virtual router using the GNS3 desktop in Windows

## Trying USB-C ethernet interface

So good news, after plugging the USB-C Ethernet interface into the laptop and cabling to the switch, in VMWare under networking, physical NICs there it is.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi19.png" alt="esxi19" width="1000px"></kbd>

## Unwinding the past
I removed the two NICs from the GNS3 VM, removed the NIC from ACCT-VM1 and then removed the 4 port groups:
 - TRUNK
 - MGMT-VLAN99
 - SALES-VLAN20
 - ACCT-VLAN10

## Setup the new network
Under Virtual Switches, Add standard virtual switch. I creatively named it vSwitch1. 

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi19.png" alt="esxi19" width="1000px"></kbd>

It could have the standard 'no' security settings.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi20.png" alt="esxi20" width="600px"></kbd>


### Setting up the Cisco switch

I plugged it into FastEthernet0/19

```
Switch#conf terminal 
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface FastEthernet0/19
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#exit
Switch#copy running-config startup
Destination filename [startup-config]? 
Building configuration...
[OK]
```

### Change the VMware management network
Management network, edit settings, change VLAN ID from 99 to 0
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi21.png" alt="esxi21" width="600px"></kbd>

At this point the connection dropped.

Now change the trunk port to VLAN 99 admin and internet
```
Switch>enable
Password: 
Switch#conf term
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface fastEthernet 0/18
Switch(config-if)#switchport access vlan 99
Switch(config-if)#switchport mode access
Switch(config-if)#no shut
Switch(config-if)#exit
Switch(config)#exit
Switch#copy running-config startup
Destination filename [startup-config]? 
Building configuration...
[OK]
```

yay it came back.

### Adding new VMWare port groups

I created dedicated GNS port groups networks because the security settings are pretty average and I wouldn't want to use them outside of GNS. Note the security settings.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi22.png" alt="esxi22" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi23.png" alt="esxi23" width="600px"></kbd>

### Add GNS-ACCT to VM ACCT-VM1
Then Add network adapter,  GNS3-ACCT

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi23.png" alt="esxi23" width="600px"></kbd>


