---
title: "Installing and configuring GNS3 on ESXi"
date: 2025-07-25
---

First I downloaded and unzip'd the ESXI image from https://gns3.com/software/download-vm

```
$ unzip GNS3.VM.VMware.ESXI.2.2.54.zip 
Archive:  GNS3.VM.VMware.ESXI.2.2.54.zip
  inflating: GNS3 VM.ova
```

switching to windows to install the client
https://docs.gns3.com/docs/getting-started/installation/windows




## router config
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
3725-01(config)#interface fastEthernet 0/0
*Mar  1 00:14:18.007: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Mar  1 00:14:19.007: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
3725-01(config)#exit
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
3725-01#show ip
```

### on stora added route for ACCT VLAN and then testing

```
root@stora:~# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.117  netmask 255.255.255.0  broadcast 192.168.30.255
root@stora:~# ip route add 10.1.10.0/255.255.255.0  via 192.168.30.5 dev eth0pipe 4
root@stora:~# ping 10.1.10.10
PING 10.1.10.10 (10.1.10.10) 56(84) bytes of data.
64 bytes from 10.1.10.10: icmp_seq=2 ttl=63 time=11.2 ms
64 bytes from 10.1.10.10: icmp_seq=3 ttl=63 time=15.6 ms
64 bytes from 10.1.10.10: icmp_seq=4 ttl=63 time=20.5 ms
64 bytes from 10.1.10.10: icmp_seq=5 ttl=63 time=14.7 ms
^C
--- 10.1.10.10 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 4030ms
rtt min/avg/max/mdev = 11.182/15.487/20.462/3.313 ms
```
### connect to raspberry pi

```
root@stora:~# ssh david@10.1.10.10
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


Last login: Wed Jul 23 19:54:35 2025 from 10.1.10.101
david@raspi:~$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.10.10  netmask 255.255.255.0  broadcast 10.1.10.255
        inet6 fe80::2ecf:67ff:fe3f:2702  prefixlen 64  scopeid 0x20<link>
```
### connect to ACCT-VM

```
root@stora:~# ssh david@10.1.10.101
The authenticity of host '10.1.10.101 (10.1.10.101)' can't be established.
ED25519 key fingerprint is SHA256:mgzDUm265QnkgZEJkPE4Di9N5F5lhZm+SCxm7L8y/RY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.1.10.101' (ED25519) to the list of known hosts.
david@10.1.10.101's password:
Linux acctvm1 6.1.0-35-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.137-1 (2025-05-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jul 25 17:46:09 2025 from 192.168.30.185
david@acctvm1:~$

```

