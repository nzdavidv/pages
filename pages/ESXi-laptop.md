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

## Boot the laptop machine from the USB drive
Here is my booting my laptop from the USB drive.
Note.. I installed to the same drive that I booted from (you can do that..)

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi1.jpg" alt="esxi1" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi2.jpg" alt="esxi2" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi3.jpg" alt="esxi3" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi4.jpg" alt="esxi4" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi5.jpg" alt="esxi5" width="900px"></kbd>

I didn't take photos of all the screens because it's a logical installer with no real surprises.
I didn't initialise the disk as part of the install.
Eventually the host came up and picked up an IP via DHCP.

## Configure ESXi

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi3.png" alt="esxi3" width="900px"></kbd>

### configure storage
In storage on the left and then devices, click on the storage, Actions, clear partition table.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi4.png" alt="esxi4" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi5.png" alt="esxi5" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi6.png" alt="esxi6" width="900px"></kbd>

Click New datastore, give it a name, use full disk.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi7.png" alt="esxi7" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi8.png" alt="esxi8" width="700px"></kbd>

### Configure networks
Networking on the left, Port groups, Add port group.
Then add the port groups for 



| VLAN Name | VLAN |
|---|---|
| ACCT-VLAN10 | 10 |
| SALES-VLAN20 | 20 |
| TRUNK | 4095 |
| MGMT-VLAN99| 99|

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi9.png" alt="esxi9" width="900px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi10.png" alt="esxi10" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi11.png" alt="esxi11" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi12.png" alt="esxi12" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi13.png" alt="esxi13" width="600px"></kbd>

### Create a VM guest

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi14.png" alt="esxi14" width="900px"></kbd>
