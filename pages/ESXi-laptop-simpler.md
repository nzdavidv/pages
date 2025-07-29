---
title: "Installing VMware ESXi 8.0.3 on an HP Elitebook laptop"
date: 2025-07-25
---
# Overview
This worked really well I have to say.. installing stock standard VMware ESXi 8.0.3 downloaded from broadcom on an HP Elitebook G5 with 16 GB RAM and 500gb NVMe drive.

## prepare USB boot drive.
In Windows with rufus..

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi0.png" alt="esxi0" width="500px"></kbd>

I said 'No' to the question..

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi0a.png" alt="esxi0a" width="500px"></kbd>

Side note.. I have tried setting up ISOs in Ubuntu and while it looks successful it's just nowhere near as reliable as good old rufus on Windows.
The Ubuntu people may throw shade at will... (or David)

## Boot the laptop machine from the USB drive
Here is my booting my laptop from the USB drive and installing ESXi
Note.. I installed to the same drive that I booted from (you can do that..).
I didn't take photos of all the screens because it's a logical installer with no real surprises.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi1.jpg" alt="esxi1" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi2.jpg" alt="esxi2" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi3.jpg" alt="esxi3" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi4.jpg" alt="esxi4" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi5.jpg" alt="esxi5" width="900px"></kbd>

I didn't initialise the disk as part of the install. This is because I wanted to confirm the network was functioning in VNWare before I committed to wiping my laptop.
Eventually the host came up and picked up an IP via DHCP.. a good sign the network is fine.

## Configure ESXi
Once it was up and running I could connect to the web interface.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi3.png" alt="esxi3" width="900px"></kbd>

### configure storage
In storage on the left and then devices, click on the storage, Actions, clear partition table.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi4.png" alt="esxi4" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi5.png" alt="esxi5" width="900px"></kbd>

Click New datastore, give it a name, use full disk.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi6.png" alt="esxi6" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi7.png" alt="esxi7" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi8.png" alt="esxi8" width="700px"></kbd>

### Upload the Guest OS ISO

To upload the ISO for the Guest VM OS deploy, click storage on the left, then the storage itself (nvme1 for me), then Datastore browser.
In there you can upload the ISO.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi13a.png" alt="esxi13a" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi13b.png" alt="esxi13b" width="600px"></kbd>

## Setup the USB-C attached network
vSwitch0 should automatically be configured.

Under Virtual Switches, Add standard virtual switch. I creatively named it vSwitch1. 

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi19.png" alt="esxi19" width="1000px"></kbd>

It could have the standard 'reject' security settings.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi20.png" alt="esxi20" width="600px"></kbd>

### Adding new VMWare port groups

I created dedicated GNS port groups networks because the security settings are pretty average and I wouldn't want to use them outside of GNS. Note the security settings.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi22.png" alt="esxi22" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi23.png" alt="esxi23" width="600px"></kbd>

### Create a guest VM

This first guest is ACCT-VM1 on ACCT-VLAN10.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi14.png" alt="esxi14" width="900px"></kbd>

Network adapter is  GNS-ACCT. Change the CD/DVD Drive 1 to Datastore ISO file and select the previously uploaded ISO 

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi27.png" alt="esxi27" width="700px"></kbd>

Configured IP manually to 10.1.10.101


Added some key packages
```
# apt install openssh-server net-tools isc-dhcp-server
```
configure DHCP
```
# vi /etc/default/isc-dhcp-server
INTERFACESv4="ens192"

# vi /etc/dhcp/dhcpd.conf
option domain-name "nzvink.com";
option domain-name-servers 8.8.8.8, 8.8.4.4; 
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
authoritative;
subnet 10.1.10.0 netmask 255.255.255.0 {
  range 10.1.10.10 10.1.10.100; # IP address range for clients
  option routers 10.1.10.1; # Default gateway
  option broadcast-address 10.1.10.255;
}

I put back /etc/network/interfaces.orig to /etc/network/interfaces so that on restart the host would have an IP on ACCT VLAN 10 network agin, then rebooted
# shutdown -h now

```

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
3725-01(config-if)#exit
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
3725-01# copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]

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


### Create a VM guest

This first guest is ACCT-VM1 on VLAN ACCT-VLAN10.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi14.png" alt="esxi14" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi15.png" alt="esxi15" width="900px"></kbd>

After this I booted it, installed Debian, configured fixed IP 10.1.10.101 after DHCP failed to get anything.
Later on will I revisit the host and temporarily connect it to VLAN 99 to install simple dhcp server. I also install VMware tools later as well.

Next blog post is <a href="Cisco-3560.md">Configuring the Cisco switch</a>



