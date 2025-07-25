---
title: "ESXi VMWare on my laptop part 2"
date: 2025-07-24
---
# Overview
Some continued config of VMWare running natively on my laptop following earlier blog. 

### recap where I am at
- ESXi is installed on my laptop but it is plugged into my home router switch not the Cisco
- The Cisco switch now has ports configured, one of them is a trunk port carrying VLANs 10,20,99
- Raspberry pi and Debian-stora plugged into the Cisco switch


## make some changes in ESXi host networking 
Now I need to change management VLAN in VMWare to 99 before unplugging the ESXi laptop from the home WiFI router and into the Cisco switch trunk port 0/18.


<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi16.png" alt="esxi16" width="500px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi17.png" alt="esxi17" width="500px"></kbd>

Then unsurprisingly when I clicked Save I lost connection. 

I physically changed ESXi host to plug into switchport 0/18 (the trunk port).

Yay - it came back.

# VM setup
in ESXi server changed ACCT-VM1 VLAN from 10 to 99 so I could connect to the web for some apt installs.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi17b.png" alt="esxi17b" width="600px"></kbd>

I need a simple dhcp server to give the raspberry pi an IP address. 

In the VM on the console after the restart (now it is on VLAN 99) I altered /etc/network/interfaces to be an IP on the home Wifi / admin network and restarted, after making a copy of /etc/network/interfaces to /etc/network/interfaces.orig

Then in VM installed some basics using console
```
# vi /etc/apt/sources.list
deb http://deb.debian.org/debian bookworm main
deb http://deb.debian.org/debian bookworm-updates main
deb http://deb.debian.org/debian-security bookworm-security main
# apt update
# apt upgrade
# apt install openssh-server net-tools
```

## VMware tools install
In VMware installed VMware tools on guest debian VM (had to eject CDROM first). yay can SSH in at least to the OS.

```
root@acctvm1:~# eject
```
then in ESXi console in the VM under Actions, guest OS, is an option to install VMware tools.
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi17c.png" alt="esxi17c" width="400px"></kbd>


Now in OS install VMware tools:
```
root@acctvm1:~# mount /dev/cdrom /mnt
mount: /mnt: WARNING: source write-protected, mounted read-only.
root@acctvm1:~# cd /mnt
root@acctvm1:/mnt# mkdir /tmp/vmware-tools
root@acctvm1:/mnt# cp -p * /tmp/vmware-tools/
root@acctvm1:/mnt# cd /tmp/vmware-tools/
root@acctvm1:/tmp/vmware-tools# eject
root@acctvm1:/tmp/vmware-tools# ls -al
total 54524
drwxr-xr-x  2 root root     4096 Jul 25 17:54 .
drwxrwxrwt 10 root root     4096 Jul 25 17:54 ..
-r-xr-xr-x  1 root root     1941 Jun 20  2023 manifest.txt
-r-xr-xr-x  1 root root     4943 Jun 20  2023 run_upgrader.sh
-r--r--r--  1 root root 53957174 Jun 20  2023 VMwareTools-10.3.26-21953278.tar.gz
-r-xr-xr-x  1 root root   917188 Jun 20  2023 vmware-tools-upgrader-32
-r-xr-xr-x  1 root root   930632 Jun 20  2023 vmware-tools-upgrader-64
root@acctvm1:/tmp/vmware-tools# ./run_upgrader.sh 
root@acctvm1:/tmp/vmware-tools# 
```
## install and configure basic DHCP server
```
# apt install isc-dhcp-server
```
..note it failed on install but I guessed this was because it didn't have a valid config so carried on 
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

I put back /etc/network/interfaces.orig to /etc/network/interfaces. Then rebooted
# shutdown -h now
```

Now change back the VLAN from 99 to 10.

After startup I can now see the Raspberry Pi which is on the same VLAN (port 0/17) in dhcp leases file.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi18.png" alt="esxi18" width="500px"></kbd>

