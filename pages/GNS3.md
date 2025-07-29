---
title: "Installing and configuring GNS3 on ESXi"
date: 2025-07-25
---

## Overview
This blog is about installing the GNS3 ESXi virtual machine appliance and configuring it.

### Recap where things are at
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi-gns-page3.png" alt="esxi-gns-page3" width="700px"></kbd>

- Cisco switch fully configured for the lab. management IP 192.168.30.188
- ESXi laptop is plugged into the trunk port 0/18 and admin network configured to VLAN 99. Management IP 192.168.30.176
- Test VM ACCT-VM1 is on ACCT VLAN 10 with a fixed IP 10.1.10.101, running a DHCP server
- Raspberry pi is on the VLAN 10 plugged into switchport 0/17 VLAN 10. It now has an IP 10.1.10.10
- Debian stora is plugged into port 0/2 on VLAN 99 IP 192.168.30.117

### Download and install the VM from OVA
First I downloaded and unzip'd the ESXI image from https://gns3.com/software/download-vm

```
$ unzip GNS3.VM.VMware.ESXI.2.2.54.zip 
Archive:  GNS3.VM.VMware.ESXI.2.2.54.zip
  inflating: GNS3 VM.ova
```

..annnnnnd this is where things went off the rails.
Rather than take it step by step through to the fail point I'll skip right ahead. 
In a nutshell trying to import the OVA into VMWare (Deploy a virtual machine from an OVF or OVA file) failed with 

> Failed - Invalid configuration for device '2'.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns0.png" alt="GNS0" width="1100px"></kbd>

I tried untar and loading the individual files (vmdk, ovf) but got the same error.

## Getting the OVA to work

```
in a tmp directory somewhere:
 $ tar xvf GNS3_VM.ova
-rw-r--r-- someone/someone 7686 2025-04-22 00:49 GNS3 VM.ovf
-rw-r--r-- someone/someone  272 2025-04-22 00:49 GNS3 VM.mf
-rw-r--r-- someone/someone 1006716416 2025-04-22 00:49 GNS3_VM-disk1.vmdk
-rw-r--r-- someone/someone    4655616 2025-04-22 00:49 GNS3_VM-disk2.vmdk
```

The modified GNS3_VM.ovf I used is at 
- [https://github.com/nzdavidv/pages/blob/main/assets/GNS3_VM.ovf](https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/assets/GNS3_VM.ovf)

```
$ cp -p GNS3_VM.ovf GNS3_VM.ovf.orig
$ wget https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/assets/GNS3_VM.ovf

..you can have a look to see the differences. I basically chopped a whole bunch out to get it to work.
$ diff GNS3_VM.ovf GNS3_VM.ovf.orig
```
### Deploy GNS3 VM from OVA

Deploy a virtual machine from an OVF or OVA file

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns1.png" alt="GNS1" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns2.png" alt="GNS2" width="600px"></kbd>

Guest OS version is Ubuntu Linux (64-bit)

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns4.png" alt="GNS4" width="800px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns3.png" alt="GNS3" width="600px"></kbd>

This takes quite a while to complete.


### Modify the GNS3 VM
Click into the VM, shut it down (I used the console and went into the menu and selected the shutdown option) and then click Edit.
Then add two network adapters of type E1000. The first one is the management network (MGMT-VLAN99) and the second the trunk network (TRUNK).

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns7.png" alt="GNS7" width="600px"></kbd>

Then expand CPU and confirm there is a tick next to Hardware Virtualization - Expose hardware assisted virtualization to the guest OS.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns6.png" alt="GNS6" width="900px"></kbd>

Now click VM Options, expand advanced, click Edit configuration

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns6a.png" alt="GNS6a" width="800px"></kbd>

Click Add parameter, down the bottom of the table click into the cell and add 
```
nestedHVEnabled
```
In the next column cell add 
```
true
```
and click OK and Save

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns6b.png" alt="GNS6b" width="800px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns6c.png" alt="GNS6c" width="800px"></kbd>

### Start the GNS3 VM

Ignore the 'SMBus host controller not enabled' on startup.
If all is well it should come up with an IP address on startup (assuming you've done the same as me and connected the management LAN to your home network wifi router which has a DHCP server)

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns8.png" alt="GNS8" width="700px"></kbd>

The website works for me but doesn't spark joy.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns8a.png" alt="GNS8a" width="1000px"></kbd>

### Configure GNS3
At this point I switch to using Windows to install the client.
- <a>https://docs.gns3.com/docs/getting-started/installation/windows</a>

In the installer I just selected the GNS3 Desktop, none of the tools etc.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns8b.png" alt="GNS8b" width="500px"></kbd>

On open of GNS3 select 'Run appliances on a remote server'

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns8c.png" alt="GNS8c" width="500px"></kbd>

Enter in the IP of the GNS3 VM, port 80, gns3 username and gns3 password.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns8d.png" alt="GNS8d" width="500px"></kbd>

Now click File, new blank project.

Click the browse routers icon and then new template.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns8e.png" alt="GNS8e" width="500px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns10.png" alt="GNS10" width="500px"></kbd>

Expand routers, click Cisco 3725 appliance, install the appliance on the main server, next, next

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns11.png" alt="GNS11" width="500px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns12.png" alt="GNS12" width="500px"></kbd>

I gave mine the name 124-25D and clicked import to browse to c3725-adventerprisek9-mz.124-25d.bin

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns13.png" alt="GNS13" width="500px"></kbd>

Drag Cisco 3725 124-25D into the big panel (Workspace).
Then click the browse all devices icon and drag the cloud into the workspace.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns14.png" alt="GNS14" width="800px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns15.png" alt="GNS15" width="400px"></kbd>

Click 'add a link', and 
- click on the Router, choosing FastEthernet 0.0 
- click on the cloud and choose Eth1 (Eth1 is the trunk port passed through, Eth0 was the admin VLAN)

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns16.png" alt="GNS16" width="700px"></kbd>

Right click on the right hand side on R1 and select 'Start'.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns17.png" alt="GNS17" width="500px"></kbd>

Mine now says telnet 192.168.30.137 5000.

## router config
Configure putty to telnet 192.168.30.137 5000

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns18.png" alt="GNS18" width="500px"></kbd>

Time to configure the router.

```
R1#enable
R1#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#hostname 3725-01
3725-01(config)#line console 0
3725-01(config-line)#logging synchronous
3725-01(config-line)#end
3725-01#
*Mar  1 00:01:54.275: %SYS-5-CONFIG_I: Configured from console by console
3725-01#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]

3725-01#conf terminal
Enter configuration commands, one per line.  End with CNTL/Z.
3725-01(config)#interface fastEthernet 0/0.10
3725-01(config-subif)#encapsulation dot1Q 10
3725-01(config-subif)#ip address 10.1.10.1 255.255.255.0
3725-01(config-subif)#no shutdown
3725-01(config-subif)#exit
3725-01(config)#interface fastEthernet 0/0.99
3725-01(config-subif)#encapsulation dot1Q 99
3725-01(config-subif)#ip address 192.168.30.5 255.255.255.0
3725-01(config-subif)#no shutdown
3725-01(config-subif)#exit
3725-01(config)#interface fastEthernet 0/0
3725-01(config-if)#no shut
3725-01(config-if)#exit
*Mar  1 00:14:18.007: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Mar  1 00:14:19.007: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
3725-01(config)#exit
3725-01#copy running-config startup-config
Destination filename [startup-config]?
*Mar  1 00:07:06.675: %SYS-5-CONFIG_I: Configured from console by console
3725-01#show ip
*Mar  1 00:14:32.431: %SYS-5-CONFIG_I: Configured from console by console
3725-01#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up
FastEthernet0/0.10         10.1.10.1       YES manual up                    up
FastEthernet0/0.99         192.168.30.5    YES manual up                    up
FastEthernet0/1            unassigned      YES unset  administratively down down
```

### on stora added route for ACCT VLAN and then testing

```
on Stora

root@stora:~# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.117  netmask 255.255.255.0  broadcast 192.168.30.255
...
root@stora:~# ip route add 10.1.10.0/255.255.255.0  via 192.168.30.5 dev eth0
```

### Connect from stora 192.168.30.117 to ACCT VM 10.1.10.101

```
on Stora

david@stora:~$ /sbin/ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.117  netmask 255.255.255.0  broadcast 192.168.30.255
...
$ ssh david@10.1.10.101
david@10.1.10.101's password:
Linux acctvm1 6.1.0-35-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.137-1 (2025-05-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Jul 26 11:00:24 2025 from 192.168.30.117
david@acctvm1:~$
logout

```
### Connect from stora 192.168.30.117 to raspberry pi 10.1.10.10

```
on Stora

david@stora:~$ /sbin/ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.117  netmask 255.255.255.0  broadcast 192.168.30.255
...

david@stora:~$ ssh david@10.1.10.10
$ ssh david@10.1.10.10
The authenticity of host '10.1.10.10 (10.1.10.10)' can't be established.
ED25519 key fingerprint is SHA256:DLb4wP9OBWv0qV48cMG76+9JcXIVfcOz3xTV0Awcy/c.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.1.10.10' (ED25519) to the list of known hosts.
david@10.1.10.10's password:
Welcome to Ubuntu 24.10 (GNU/Linux 6.11.0-1015-raspi aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

2 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Failed to connect to https://changelogs.ubuntu.com/meta-release. Check your Internet connection or proxy settings
david@raspi:~$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.10.10  netmask 255.255.255.0  broadcast 10.1.10.255
        inet6 fe80::2ecf:67ff:fe3f:2702  prefixlen 64  scopeid 0x20<link>
```

### Yay! it's all working
Recap

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi-gns-drawio.png" alt="esxi-gns-drawio" width="700px"></kbd>
- Cisco switch fully configured for the lab. management IP 192.168.30.188
- ESXi laptop is plugged into the trunk port 0/18 and admin network configured to VLAN 99. Management IP 192.168.30.176
- Test VM ACCT-VM1 is on ACCT VLAN 10 with a fixed IP 10.1.10.101, running a DHCP server. It has default route 10.1.10.1.
- Raspberry pi is on the VLAN 10 plugged into switchport 0/17 VLAN 10. It has IP 10.1.10.10 (issued by ACCT-VM1 DHCP server) and default route 10.1.10.1.
- GNS3 VM deployed with 2 interfaces of type E1000. The first is the management network (MGMT-VLAN99) and the second the trunk network (TRUNK). The GNS3 management IP is 192.168.30.139
- A GNS config deployed with virtual router, the router has 2 dot1Q interfaces. One on VLAN 10 with IP 10.1.10.1 and one on VLAN99 with IP 192.168.30.5.
- Debian stora is plugged into port 0/2 on VLAN 99 IP 192.168.30.117. It has a static route to get to 10.1.10.0/24 via 192.168.30.5

<img src="https://octodex.github.com/images/welcometocat.png" height="200px" />

to-do - see if I can figure a way to get ACCT VLAN 10 to connect to the internet.


