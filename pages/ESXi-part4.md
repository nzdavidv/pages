### Adding USB NVMe drive

I thought this would be an amazing  idea.. adding NVMe drive.

It wasn't a bad idea but was a lot more complex than I thought.

I followed this guide <a>https://sysadmin102.com/2025/02/how-to-use-a-usb-drive-as-a-vmfs-datastore-in-vmware-esxi-step-by-step-guide/</a>

To enable SSH.. Host, Actions, Services, enable SSH

Then SSH in as root.

```

login as: root

[root@localhost:~] /etc/init.d/usbarbitrator stop
stopping usbarbitrator...
usbarbitrator stopped
[root@localhost:~] chkconfig usbarbitrator off
[root@localhost:~] ls -lh /dev/disks
total 1187109536
-rw-------    1 root     root       29.5G Aug  9 02:33 mpx.vmhba32:C0:T0:L0
-rw-------    1 root     root      100.0M Aug  9 02:33 mpx.vmhba32:C0:T0:L0:1
-rw-------    1 root     root        4.0G Aug  9 02:33 mpx.vmhba32:C0:T0:L0:5
-rw-------    1 root     root        4.0G Aug  9 02:33 mpx.vmhba32:C0:T0:L0:6
-rw-------    1 root     root       21.4G Aug  9 02:33 mpx.vmhba32:C0:T0:L0:7
-rw-------    1 root     root      119.2G Aug  9 02:33 mpx.vmhba33:C0:T0:L0
-rw-------    1 root     root       16.0M Aug  9 02:33 mpx.vmhba33:C0:T0:L0:1
-rw-------    1 root     root      476.9G Aug  9 02:33 t10.NVMe____KXG5AZNV512G_TOSHIBA____________________77F55800020D0800
-rw-------    1 root     root      100.0M Aug  9 02:33 t10.NVMe____KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:1
-rw-------    1 root     root       16.0M Aug  9 02:33 t10.NVMe____KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:2
-rw-------    1 root     root      224.0G Aug  9 02:33 t10.NVMe____KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:3
-rw-------    1 root     root      650.0M Aug  9 02:33 t10.NVMe____KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:4
-rw-------    1 root     root      252.2G Aug  9 02:33 t10.NVMe____KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:5
lrwxrwxrwx    1 root     root          20 Aug  9 02:33 vml.01000000003031323334353637383932334745494c2055 -> mpx.vmhba33:C0:T0:L0
lrwxrwxrwx    1 root     root          22 Aug  9 02:33 vml.01000000003031323334353637383932334745494c2055:1 -> mpx.vmhba33:C0:T0:L0:1
lrwxrwxrwx    1 root     root          20 Aug  9 02:33 vml.01000000003033313035393038414441544120 -> mpx.vmhba32:C0:T0:L0
lrwxrwxrwx    1 root     root          22 Aug  9 02:33 vml.01000000003033313035393038414441544120:1 -> mpx.vmhba32:C0:T0:L0:1
lrwxrwxrwx    1 root     root          22 Aug  9 02:33 vml.01000000003033313035393038414441544120:5 -> mpx.vmhba32:C0:T0:L0:5
lrwxrwxrwx    1 root     root          22 Aug  9 02:33 vml.01000000003033313035393038414441544120:6 -> mpx.vmhba32:C0:T0:L0:6
lrwxrwxrwx    1 root     root          22 Aug  9 02:33 vml.01000000003033313035393038414441544120:7 -> mpx.vmhba32:C0:T0:L0:7
lrwxrwxrwx    1 root     root          68 Aug  9 02:33 vml.0100000000373746355f353830305f303230445f30383030004b584735415a -> t10.NVMe____KXG5AZ                                           NV512G_TOSHIBA____________________77F55800020D0800
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0100000000373746355f353830305f303230445f30383030004b584735415a:1 -> t10.NVMe____KXG5                                           AZNV512G_TOSHIBA____________________77F55800020D0800:1
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0100000000373746355f353830305f303230445f30383030004b584735415a:2 -> t10.NVMe____KXG5                                           AZNV512G_TOSHIBA____________________77F55800020D0800:2
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0100000000373746355f353830305f303230445f30383030004b584735415a:3 -> t10.NVMe____KXG5                                           AZNV512G_TOSHIBA____________________77F55800020D0800:3
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0100000000373746355f353830305f303230445f30383030004b584735415a:4 -> t10.NVMe____KXG5                                           AZNV512G_TOSHIBA____________________77F55800020D0800:4
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0100000000373746355f353830305f303230445f30383030004b584735415a:5 -> t10.NVMe____KXG5                                           AZNV512G_TOSHIBA____________________77F55800020D0800:5
lrwxrwxrwx    1 root     root          68 Aug  9 02:33 vml.0554b3c4d345f453a7a6065680b7a15359750242bde2cf35ba89011211ca8b9e41 -> t10.NVMe____KX                                           G5AZNV512G_TOSHIBA____________________77F55800020D0800
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0554b3c4d345f453a7a6065680b7a15359750242bde2cf35ba89011211ca8b9e41:1 -> t10.NVMe____                                           KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:1
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0554b3c4d345f453a7a6065680b7a15359750242bde2cf35ba89011211ca8b9e41:2 -> t10.NVMe____                                           KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:2
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0554b3c4d345f453a7a6065680b7a15359750242bde2cf35ba89011211ca8b9e41:3 -> t10.NVMe____                                           KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:3
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0554b3c4d345f453a7a6065680b7a15359750242bde2cf35ba89011211ca8b9e41:4 -> t10.NVMe____                                           KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:4
lrwxrwxrwx    1 root     root          70 Aug  9 02:33 vml.0554b3c4d345f453a7a6065680b7a15359750242bde2cf35ba89011211ca8b9e41:5 -> t10.NVMe____                                           KXG5AZNV512G_TOSHIBA____________________77F55800020D0800:5

-- mpx.vmhba33:C0:T0:L0 is my USB attached NVMe drive--

[root@localhost:~] partedUtil mklabel /dev/disks/mpx.vmhba33:C0:T0:L0 gpt
[root@localhost:~] partedUtil getptbl /dev/disks/mpx.vmhba33:C0:T0:L0
gpt
15566 255 63 250069680

sectors is (15566 * 255 * 63) -1

[root@localhost:~] # 250067789
[root@localhost:~] partedUtil setptbl /dev/disks/mpx.vmhba33:C0:T0:L0 gpt "1 2048 250067789 AA31E02A400F11DB9590000C2911D1B8 0"
gpt
0 0 0 0
1 2048 250067789 AA31E02A400F11DB9590000C2911D1B8 0

--before view
[root@localhost:~] esxcli storage filesystem list
Mount Point                                        Volume Name                                 UUID                                 Mounted  Type           Size         Free
-------------------------------------------------  ------------------------------------------  -----------------------------------  -------  ------  -----------  -----------
/vmfs/volumes/688129d7-e429fe4e-7e65-c46516f1f5b5  LOCKER-688129d7-e429fe4e-7e65-c46516f1f5b5  688129d7-e429fe4e-7e65-c46516f1f5b5     true  VMFSOS  22817013760  21341667328
/vmfs/volumes/639cd5f2-c248be29-26be-3f6e81a98ebc  BOOTBANK1                                   639cd5f2-c248be29-26be-3f6e81a98ebc     true  vfat     4293591040   4021813248
/vmfs/volumes/cd9c4c29-a840b718-ed9e-ced5ee0de2b2  BOOTBANK2                                   cd9c4c29-a840b718-ed9e-ced5ee0de2b2     true  vfat     4293591040   4293328896

-- note the :1 at the end - presumably for the partition --

[root@localhost:~] vmkfstools -C vmfs6 -S USB-Datastore /dev/disks/mpx.vmhba33:C0:T0:L0:1
create fs deviceName:'/dev/disks/mpx.vmhba33:C0:T0:L0:1', fsShortName:'vmfs6', fsName:'USB-Datastore'
deviceFullPath:/dev/disks/mpx.vmhba33:C0:T0:L0:1 deviceFile:mpx.vmhba33:C0:T0:L0:1
ATS on device /dev/disks/mpx.vmhba33:C0:T0:L0:1: not supported
.
Checking if remote hosts are using this device as a valid file system. This may take a few seconds...
Creating vmfs6 file system on "mpx.vmhba33:C0:T0:L0:1" with blockSize 1048576, unmapGranularity 1048576, unmapPriority default and volume label "USB-Datastore".
Successfully created new volume: 6896b7e6-94bc1a70-ad35-c46516f1f5b5

-- after view. new datastore 'USB-Datastore'
[root@localhost:~] esxcli storage filesystem list
Mount Point                                        Volume Name                                 UUID                                 Mounted  Type            Size          Free
-------------------------------------------------  ------------------------------------------  -----------------------------------  -------  ------  ------------  ------------
/vmfs/volumes/6896b7e6-94bc1a70-ad35-c46516f1f5b5  USB-Datastore                               6896b7e6-94bc1a70-ad35-c46516f1f5b5     true  VMFS-6  127775277056  126259036160
/vmfs/volumes/688129d7-e429fe4e-7e65-c46516f1f5b5  LOCKER-688129d7-e429fe4e-7e65-c46516f1f5b5  688129d7-e429fe4e-7e65-c46516f1f5b5     true  VMFSOS   22817013760   21341667328
/vmfs/volumes/639cd5f2-c248be29-26be-3f6e81a98ebc  BOOTBANK1                                   639cd5f2-c248be29-26be-3f6e81a98ebc     true  vfat      4293591040    4021813248
/vmfs/volumes/cd9c4c29-a840b718-ed9e-ced5ee0de2b2  BOOTBANK2                                   cd9c4c29-a840b718-ed9e-ced5ee0de2b2     true  vfat      4293591040    4293328896
[root@localhost:~] vmkfstools -C vmfs6 -S USB-Datastore /dev/disks/mpx.vmhba33:C0:T0:L0:1^C

```
