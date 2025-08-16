---
title: "Installing oracle linux on Raspberry Pi"
date: 2025-08-16
---
# Overview
Downloading and installing oracle linux on raspberry pi (raspi for short).

- Oracle Linux 9
- Raspberry Pi 4 Model B Rev 1.2

It doesn't work on Raspiperry Pi v5 but my older one is fine.

Following <a>https://docs.oracle.com/en/learn/oracle-linux-install-rpi</a>

## prep the memory card
Fortunately I had a spare SD I could use to give me a backout plan if this all goes pear shaped.


For some reason the download from oracle didn't go well but I could download the wget file.
```
[ on my laptop using ubuntu ]
$ chmod 755 wget.sh 
$ ./wget.sh

I unzip'd the resulting zip file

$ unzip V1050512-01.zip 
Archive:  V1050512-01.zip
  inflating: rpi-ol9.6-image-20250531.img.xz

and unmounted my micro SD and loaded the image on it

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
...
/dev/mmcblk0p1   30G   32K   30G   1% /media/david/6163-3638

$ umount /media/david/6163-3638 
$ sudo su -
# xzcat rpi-ol9.6-image-20250531.img.xz > /dev/mmcblk0

```

## fire it up 
Turning on raspi..

ok so SUPER ANNOYING that you can't ssh in as root to begin with. I had to connect a monitor and keyboard (fortunately I had an HDMI to mini-HDMI or whatever it is called).
After that yep could login with root and oracle and add my user account.

Now I can SSH in.
```
david@david-Modern-14-C12M:~/Downloads$ ssh david@raspidev.nzvink.com
The authenticity of host 'raspidev.nzvink.com (192.168.10.100)' can't be established.
ED25519 key fingerprint is SHA256:hZ6KvEVLNiU7u5SCcep71hhuJeqxtt/kU+Fz3RJ+1Uo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'raspidev.nzvink.com' (ED25519) to the list of known hosts.
david@raspidev.nzvink.com's password: 
Last login: Sat Aug 16 03:15:33 2025 from 192.168.10.101
```
## Configure the OS

### Resize the root filesystem
```
[david@rpi ~]$ su -
Password: 
Last login: Sat Aug 16 03:15:38 UTC 2025 on pts/0
Last failed login: Sat Aug 16 03:21:26 UTC 2025 on pts/1
[root@rpi ~]# mount|grep root
/dev/mmcblk0p3 on / type btrfs (rw,relatime,seclabel,ssd,discard=async,space_cache=v2,subvolid=257,subvol=/root)
[root@rpi ~]# growpart /dev/mmcblk0 3
CHANGED: partition=3 start=788480 old: size=6815744 end=7604223 new: size=61545439 end=62333918
[root@rpi ~]# btrfs filesystem resize max /
Resize device id 1 (/dev/mmcblk0p3) from 3.25GiB to max
```

### Install core software
```
# yum update
# yum install httpd php  php-gd php-xml mariadb-server php-mbstring php-json wget tar lshw

-- install php modules
# dnf install php-mysqli php-intl
```
### enable webserver
```
# systemctl enable httpd
--firewall? apparently not
```

### install and config database

```
# systemctl start mariadb
# systemctl enable mariadb

# mysql_secure_installation

Enter current password for root (enter for none): [enter]
OK, successfully used password, moving on...

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!

Remove anonymous users? [Y/n] y
 ... Success!

Disallow root login remotely? [Y/n] y
 ... Success!

Remove test database and access to it? [Y/n] y

Reload privilege tables now? [Y/n] y

#  mysql -u root -p
MariaDB [(none)]>
CREATE USER 'mwsql'@'localhost' IDENTIFIED BY 'password-in-password-store';
CREATE DATABASE my_wiki;
GRANT ALL PRIVILEGES ON my_wiki.* TO 'mwsql'@'localhost' WITH GRANT OPTION;
```
### get mediawiki files 
```
# cd /var/www/html
# wget https://releases.wikimedia.org/mediawiki/1.44/mediawiki-1.44.0.tar.gz
# gunzip mediawiki-1.44.0.tar.gz   ; tar xvf mediawiki-1.44.0.tar
# chown -R apache:apache mediawiki-1.44.0/
# ln -s mediawiki-1.44.0/ mediawiki
```

### transfer and load mediawiki content from raspberry pi
```
[on raspi]
--tar up mediawiki images--
# cd /var/www/html/mediawiki; tar cvf images.tar images
--transfer to oralinux01
# scp images.tar /var/www/html/mediawiki/resources/assets/mw-logo-135.jpg david@192.168.30.126:/tmp
--copy over config file to oralinux01
# scp LocalSettings.php david@192.168.30.126:/tmp
--copy database dump
# mysqldump --user=root -p --host=localhost my_wiki > /tmp/my_wiki.sql
# scp my_wiki.sql david@192.168.30.126:

[on oralinux01]
-- put the files in place
# cd /var/www/html/mediawiki/
# tar xvf /tmp/images.tar
# cp /tmp/mw-logo-135.jpg /var/www/html/mediawiki/resources/assets/
# cp /tmp/LocalSettings.php .
# vi LocalSettings.php
--update $wgDBpassword
--update $wgServer

--set file permissions and load database from backup
# cd /var/www
# chown -R apache:apache /var/www/html
# cd ~david
--oralinux01 mariadb couldn't deal with the collation from the raspberry pi (oralinux01 is an older version)
# sed -i 's/utf8mb4_0900_ai_ci/utf8mb4_unicode_ci/g' my_wiki.sql 
# mysql -p < my_wiki.sql my_wiki

--restart DB and webserver
# systemctl restart mariadb
# systemctl restart httpd
```

### reinstall php
turns out the PHP that shipped with ora linux 9 is too old.
```
# dnf install epel-release -y
# yum remove php
# dnf module list php
# dnf install @php:8.3
# systemctl restart httpd
```

### enabling uploads in mediawiki
```
# cp -p /etc/php.ini /etc/php.ini-2025-08-16
# vi /etc/php.ini
upload_tmp_dir = /var/www/html/upload-tmp-dir 
# mkdir /var/www/html/upload-tmp-dir ; chown apache:apache /var/www/html/upload-tmp-dir
# systemctl restart httpd

# semanage fcontext -a -t httpd_sys_script_exec_t /var/www/html/mediawiki-1.44.0/includes/GlobalFunctions.php
# semanage fcontext -a -t httpd_tmp_t "/var/www/html/upload-tmp-dir(/.*)?"
# semanage fcontext -a -t httpd_user_rw_content_t "/var/www/html/mediawiki-1.44.0/images(/.*)?"
# restorecon -vR /var/www/html/mediawiki-1.44.0/

# cat /etc/selinux/targeted/contexts/files/file_contexts.local
..
/var/www/html/mediawiki-1.44.0/includes/GlobalFunctions.php    system_u:object_r:httpd_sys_script_exec_t:s0
/var/www/html/upload-tmp-dir(/.*)?    system_u:object_r:httpd_tmp_t:s0
/var/www/html/mediawiki-1.44.0/images(/.*)?    system_u:object_r:httpd_user_rw_content_t:s

```

### testing

yup all worked fine

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/upload-win.png" alt="upload-win"  width="800px"></kbd>
