---
title: "Installing VMware ESXi 8.0.3 on an HP Elitebook laptop"
date: 2025-07-25
---
# Overview
This worked really well I have to say.. installing stock standard VMware ESXi 8.0.3 downloaded from broadcom on an HP Elitebook G5 with 16 GB RAM and 500gb NVMe drive.

## initial setup
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi-gns-simpler2.png" alt="esxi-gns-simpler2" width="700px"></kbd>

The initial setup has:
- the switch with admin and internet network. Network is the same as my home WiFi network - 192.168.30.0/24
  - Home WiFi router is plugged in
  - Debian-Stora plugged in with IP 192.168.30.117
  - The ESXi laptop is plugged in with its onboard Ethernet port. It has a DHCP reservation for IP 192.168.30.176
 
- the switch with 'ACCT' network. Yet to be defined.
  - The ESXi laptop is plugged in with its USB-C adapter Ethernet port.
  - Raspberry Pi is plugged into this switch. Initially it has no IP address. 

# ESXi laptop setup
## prepare USB boot drive.
In Windows with rufus..

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi0.png" alt="esxi0" width="500px"></kbd>

I said 'No' to the question..

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi0a.png" alt="esxi0a" width="500px"></kbd>

Side note.. I have tried setting up ISOs in Ubuntu and while it looks successful it's just nowhere near as reliable as good old rufus on Windows.
The Ubuntu people may throw shade at will... (or David)

## Boot the laptop from the USB drive
Here is my booting my laptop from the USB drive and installing ESXi
Note.. I installed to the same drive that I booted from (you can do that..).
I didn't take photos of all the screens because it's a logical installer with no real surprises.
Taking photos of screens makes me die a little inside but I don't think there is any otherway when booting an installer.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi1.jpg" alt="esxi1" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi2.jpg" alt="esxi2" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi3.jpg" alt="esxi3" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi4.jpg" alt="esxi4" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi5.jpg" alt="esxi5" width="900px"></kbd>

I didn't initialise the disk as part of the install. This is because I wanted to confirm the network was functioning in VMWare before I committed to wiping my laptop.
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

I created dedicated GNS port groups. Note the security settings.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi22.png" alt="esxi22" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi23.png" alt="esxi23" width="600px"></kbd>

### Create a guest VM

This first guest is ACCT-VM1 on network GNS-MGMT. After it is built and has the DHCP configured I will change it to GNS-ACCT network.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi14.png" alt="esxi14" width="900px"></kbd>

Network adapter is  GNS-ACCT. Change the CD/DVD Drive 1 to Datastore ISO file and select the previously uploaded ISO 

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi28.png" alt="esxi28" width="700px"></kbd>

Add openssh-server from the console as part of install.

Then I can connect via SSH and add the other software, configure DHCP, then shutdown the VM
```
# apt install net-tools isc-dhcp-server
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
```
Now setup the VM for fixed IP boot on the ACCT network
```
# vi /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens192
iface ens192 inet static
        address 10.1.10.101/24
        gateway 10.1.10.1
        # dns-* options are implemented by the resolvconf package, if installed
        dns-nameservers 10.1.10.1

# shutdown -h now
```

### Change ACCT-VM1 network in VMWare
Now in VMWare I can change the VM network to GNS-ACCT network

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi29.png" alt="esxi29" width="700px"></kbd>

Then start the server again.

### Basic test that the ACCT network is working

I started the VM and initially it couldn't ping the raspberry pi but then it came right (could be DHCP on the pi needed the VM DHCP service to be up and it took a while).

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi25.png" alt="esxi25" width="600px"></kbd>

# GNS3 install
### Download and install the VM OVA
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

Deploy a virtual machine from an OVF or OVA file. The OVA file is the modified one.. nothing else is changed.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns1.png" alt="GNS1" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns2.png" alt="GNS2" width="600px"></kbd>

Guest OS version is Ubuntu Linux (64-bit)

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns4.png" alt="GNS4" width="800px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns3.png" alt="GNS3" width="600px"></kbd>

This takes quite a while to complete.


### Modify the GNS3 VM
Click into the VM, shut it down (I used the console and went into the menu and selected the shutdown option) and then click Edit.
Then add two network adapters of type E1000. The first one is the GNS-mgmt and the second GNS-ACCT.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns20.png" alt="GNS20" width="600px"></kbd>

Then expand CPU and confirm there is a tick next to Hardware Virtualization - Expose hardware assisted virtualization to the guest OS.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns6.png" alt="GNS6" width="800px"></kbd>

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

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns21.png" alt="GNS21" width="700px"></kbd>

The website works for me but doesn't spark joy.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns8a.png" alt="GNS8a" width="1000px"></kbd>

## Install GNS3 desktop

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

Click Create a new version.
I gave mine the name 124-25D and clicked import to browse to c3725-adventerprisek9-mz.124-25d.bin

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns13.png" alt="GNS13" width="500px"></kbd>

Drag Cisco 3725 124-25D into the big panel (Workspace).
Then click the browse all devices icon and drag the cloud into the workspace.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns14.png" alt="GNS14" width="800px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns15.png" alt="GNS15" width="400px"></kbd>

Then add one more Cloud.
 - I named the first one admin-net and the second acct-net.
 - I added a link from admin-net (eth0) to the router fa0/0.
 - I added a link from acct-net (eth1) to the router fa0/1.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns19.png" alt="gns19" width="1000px"></kbd>

Then click the start green triangle in the toolbar.

## Configure the virtual router
### Connect to the virtual router
Now use putty to telnet to the virtual router with the details from R1 on the right hand side.
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/gns22.png" alt="gns22" width="700px"></kbd>

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



