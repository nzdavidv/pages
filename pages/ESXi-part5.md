## Adding an NFS mount.

ok, this probably sounds crazy but.. I needed more storage and thought I could use NFS.

I tried NFS to my old NetGear stora running Debian but it was so slow it was unusable.

Just rank. I don't know why it was so bad but it was.


So then, and ok.. this is unorthodox but I used my Raspberry Pi (Raspberry Pi 5 Model B Rev 1.0) with an SSD attached by USB.

It's not awful.

### Creating the NFS share
On the Raspberry Pi
```
# apt install nfs-kernel-server
# mkdir -p /mnt/NFS
# chown nobody:nogroup /mnt/NFS/
# chmod 777 /mnt/NFS/
# vi /etc/exports
/mnt/NFS 192.168.30.176(rw,sync,no_subtree_check,no_root_squash)

# systemctl restart nfs-kernel-server

--confirming it can to v4
# cat /proc/fs/nfsd/versions
+3 +4 +4.1 +4.2

```

### configuring in ESXi
- storage, new datastore
- mount NFS datastore
- add the name, IP, and the NFS share path, v4

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi-nfs1.png" alt="esxi1" width="500px"></kbd>
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/esxi-nfs2.png" alt="esxi2" width="500px"></kbd>

done. all too easy.

As a test I moved a 16GB VM onto it and it wasn't too slow for me.
