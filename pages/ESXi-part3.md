---
title: "ESXi using USB-C ethernet"
date: 2025-07-28
---
# Overview
Continued fun in the lab but instead of using VLAN tagging using two seperate NICs.
One of them is USB-C ethernet adapter, plugged into the laptop running ESXi.

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

It could have the standard 'reject' security settings.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi20.png" alt="esxi20" width="600px"></kbd>


### Setting up the Cisco switch

I plugged the ethernet cable from the laptop USB-C ethernet adapter into the Cisco switch - FastEthernet0/19.
This is me configuring the switch port to VLAN10.

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

At this point the connection to the VMWare host web interface dropped.

Now I change fastEthernet 0/18 from the trunk port to VLAN 99 (admin and internet)
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

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi24.png" alt="esxi24" width="600px"></kbd>

I started the VM and initially it couldn't ping the raspberry pi but then it came right (could be DHCP on the pi needed the VM DHCP service to be up and it took a while).

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi25.png" alt="esxi25" width="600px"></kbd>

### Add the new networks to GNS3 VM

In VMWare edit the GNS3 VM, Add network adapter for the new Management network first then Add network adapter GNS-ACCT.

Remember to make them E1000 adapter types.

Now start the VM and it's time to switch to using Windows to configure GNS3.

The admin interface came up 192.68.30.139 which is good.

### Configure GNS3
In Windows GNS3 desktop (when I can connected to the correct WiFi network) I dragged the router on the Workspace, and then two clouds.

 - I named the first one admin-net and the second acct-net.
 - I added a link from admin-net (eth0) to the router fa0/0.
 - I added a link from acct-net (eth1) to the router fa0/1.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns19.png" alt="gns19" width="1000px"></kbd>

Then click the start green triangle in the toolbar.

### Configure the virtual router

```
R1#enable
R1#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#hostname 3725-01
3725-01(config)#line console 0
3725-01(config-line)#logging synchronous
3725-01(config-line)#end
3725-01#
*Mar  1 00:03:56.651: %SYS-5-CONFIG_I: Configured from console by console

3725-01#conf term
Enter configuration commands, one per line.  End with CNTL/Z.
3725-01(config)#interface fastEthernet 0/0
3725-01(config-if)#ip address 192.168.30.30 255.255.255.0
3725-01(config-if)#no shut
*Mar  1 00:06:19.339: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Mar  1 00:06:20.339: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
3725-01(config-if)#interface fastEthernet 0/1
3725-01(config-if)#ip address 10.1.10.1 255.255.255.0
3725-01(config-if)#no shut
3725-01(config-if)#
*Mar  1 00:06:50.103: %LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to up
*Mar  1 00:06:51.103: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed

3725-01(config-if)#exit
3725-01(config)#exit
3725-01#
*Mar  1 00:08:01.063: %SYS-5-CONFIG_I: Configured from console by console
3725-01#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.30.30   YES manual up                    up
FastEthernet0/1            10.1.10.1       YES manual up                    up
3725-01#
*Mar  1 00:15:50.527: %SYS-5-CONFIG_I: Configured from console by console

3725-01#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.30.0/24 is directly connected, FastEthernet0/0
     10.0.0.0/24 is subnetted, 1 subnets
C       10.1.10.0 is directly connected, FastEthernet0/1

```

## testing
Add the route to my stora and test

```
root@stora:~# ip route add 10.1.10.0/255.255.255.0  via 192.168.30.30 dev eth0
root@stora:~# 
david@stora:~$ /sbin/ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.117  netmask 255.255.255.0  broadcast 192.168.30.255
david@stora:~$ ssh david@10.1.10.101
david@10.1.10.101's password:
Linux acctvm1 6.1.0-35-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.137-1 (2025-05-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Jul 27 03:50:42 2025 from 192.168.30.117
```

## Success!
<img src="https://octodex.github.com/images/welcometocat.png" height="200px" />
