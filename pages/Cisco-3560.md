---
title: "Cisco 3560 switch setup"
date: 2025-07-24
---
# Overview
This is my notes for setting up a Cisco 3560 switch someone gave me.
I'm not a network engineer, have never been a network engineer.. and may be doing things incorrectly / there may be better ways to do this.


## Serial console setup
To setup I connected my daily laptop running windows with USB serial port into a Cisco serial to console cable.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/cisco0.jpg" alt="cisco-0" width="500px"></kbd>

Had a look in device manager and saw the USB serial port was com3.
Putty COM3 9600

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/cisco1.png" alt="cisco-1" width="500px"></kbd>

I know I could use minicom and Ubuntu but honestly again Windows is just easier for some things. (In my Netgear stora blog I use minicom on the raspberry pi to connect to the stora console out of necessity).

## Factory reset the switch 
Probably I'm using the wrong words..  but first step was to wipe the config on the switch to start clean.

```
Loading "flash:c3560-ipbase-mz.122-35.SE5/c3560-ipbase-mz.122-35.SE5.bin"...@@@@                                 	@
File "flash:c3560-ipbase-mz.122-35.SE5/c3560-ipbase-mz.122-35.SE5.bin" uncompressed and installed, entry point: 0x3000
executing...
 
..Press RETURN to get started!
 
 
00:00:37: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to down
00:00:37: %SPANTREE-5-EXTENDED_SYSID: Extended SysId enabled for type vlan
00:00:59: %SYS-5-RESTART: System restarted --
 
Would you like to terminate autoinstall? [yes]:
 
     	--- System Configuration Dialog ---
 
Would you like to enter the initial configuration dialog? [yes/no]:
% Please answer 'yes' or 'no'.
Would you like to enter the initial configuration dialog? [yes/no]: no
Switch>
Switch>
 
Switch>enable
Switch#config
Switch#write erase
Erasing the nvram filesystem will remove all configuration files! Continue? [confirm]y[OK]
Erase of nvram: complete
Switch#
00:06:10: %SYS-7-NV_BLOCK_INIT: Initialized the geometry of nvram
Switch#delete flash:vlan.dat
Delete filename [vlan.dat]?
Delete flash:vlan.dat? [confirm]y
%Error deleting flash:vlan.dat (No such file or directory)
Switch#
Switch#reload

System configuration has been modified. Save? [yes/no]: yes
Building configuration...
[OK]
Proceed with reload? [confirm]
 
...done Initializing Flash.
Boot Sector Filesystem (bs) installed, fsid: 3
done.
```

## Setting up for telnet
[ VLAN 99 is connecting to my home internet router. ] 

Next setup passwords and configure VLAN 99 

 ```
Switch>enable
Switch#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface fastEthernet 0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 99
% Access VLAN does not exist. Creating vlan 99
Switch(config-if)#exit
Switch(config)#interface vlan 99
Switch(config-if)#ip address 192.168.30.188 255.255.255.0
Switch(config-if)#no shutdown
Switch(config-if)#
Switch(config-if)#exit
 
Switch(config)#exit
Switch#line console 0
Switch#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#line console 0
Switch(config-line)#password secretpassword
Switch(config-line)#login
Switch(config-line)#exit
Switch(config)#line vty 0 4
Switch(config-line)#password secretpassword
Switch(config-line)#exit
Switch(config)#exit
Switch#show vlan
 
VLAN Name                             Status	Ports
---- -------------------------------- --------- -------------------------------
1	default                      	active	Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/18, Fa0/19, Fa0/20, Fa0/21
                                                Fa0/22, Fa0/23, Fa0/24, Fa0/25
                                                Fa0/26, Fa0/27, Fa0/28, Fa0/29
                                                Fa0/30, Fa0/31, Fa0/32, Fa0/33
                                                Fa0/34, Fa0/35, Fa0/36, Fa0/37
                                                Fa0/38, Fa0/39, Fa0/40, Fa0/41
                                                Fa0/42, Fa0/43, Fa0/44, Fa0/45
                                                Fa0/46, Fa0/47, Fa0/48, Gi0/1
                                                Gi0/2, Gi0/3, Gi0/4
99   VLAN0099                     	active	Fa0/1
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
 
VLAN Type  SAID   	MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
 
Switch#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#enable secret secretpassword
Switch(config)#exit
Switch#copy running-config startup
Destination filename [startup-config]?
Building configuration...
[OK]
Switch#
```
 
After all that plugged port Fa 0/1 into my home WiFi router switch and
Could telnet to 192.168.30.188

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/cisco2.png" alt="aws-cisco-2" width="500px"></kbd>

## Configure Cisco switch ports

Next I setup a port for stora [port 0/2 - VLAN 99]
```
Switch>enable
Switch#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface fastEthernet 0/2
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 99
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#exit
Switch#copy running-config startup
Destination filename [startup-config]? 
Building configuration...
[OK]
```

then raspberry pi [port 0/17 - VLAN 10]
```
Switch>enable
Switch#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)# interface FastEthernet0/17
Switch(config-if)#switchport access vlan 10
Switch(config-if)#exit
Switch(config)#exit
Switch#copy running-config startup
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Then the trunk port [ port 0/18 - VLANs 10,99 ]
```
Switch>enable
Switch#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface fastEthernet 0/18
Switch(config-if)#switchport trunk encapsulation dot1q
Switch(config-if)#switchport mode trunk               
Switch(config-if)#switchport trunk allowed vlan 10,99
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#exit
Switch# copy running-config startup
Destination filename [startup-config]? 
Building configuration...
[OK]
```

## plugging devices into the switch
- Plugged stora into 0/2 (and can connect to it)
- Plugged raspberry pi into 0/17 (and can't connect to it)

### Recap where things are at
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi-gns-page2.png" alt="esxi-gns-page2" width="700px"></kbd>

- installed ESXi onto the laptop.
- Laptop is stil plugged into home WiFi router.
- Cisco switch is configured

Next blog post is <a href="ESXi-part2.md">Revisit VMware before moving on to GNS3</a>



