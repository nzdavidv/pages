## Oracle Linux lab

Impressed with oracle Linux 10.
SELinux isn't fun but once you get the hang of it maybe is ok.

to explore later..
<a>https://docs.oracle.com/en/learn/oracle-linux-install-rpi/#introduction</a>

I installed it as a VM on ESXi with 4GB RAM and 2x VCPUs and it hums.

Now going to install MediaWiki hopefully, loosely following:
- <a>https://dev.to/nhisyamj/installing-apache-php-and-mysql-on-oracle-linux-8-3j7m</a>
- <a>https://www.mediawiki.org/wiki/Manual:Installing_MediaWiki</a>


### install packages
```
# yum install httpd php  php-gd php-xml mariadb-server php-mbstring php-json
```
### get webserver running
```
# systemctl enable httpd

--config firewall
# firewall-cmd --add-service=http --permanent
# firewall-cmd --reload

-- install php modules
# dnf install php-mysqli php-intl
```

### database

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
### get mediawiki files and database backup 
```
# cd /var/www/html
# wget https://releases.wikimedia.org/mediawiki/1.44/mediawiki-1.44.0.tar.gz
# gunzip mediawiki-1.44.0.tar.gz   ; tar xvf mediawiki-1.44.0.tar
# chown -R apache:apache mediawiki-1.44.0/
# ln -s mediawiki-1.44.0/ mediawiki
```

### restoring from raspidev
```
--on raspi tar up mediawiki images--
root@raspi:/var/www/html/mediawiki# tar cvf images.tar images
# scp images.tar david@192.168.30.126:/tmp
--also copy over config file
# scp LocalSettings.php david@192.168.30.126:/tmp
--database dump
# mysqldump --user=root -p --host=localhost my_wiki > /tmp/my_wiki.sql
# scp my_wiki.sql david@192.168.30.126:

--on oralinux01 
# cd /var/www/html/mediawiki/
# tar xvf /tmp/images.tar
# cp /tmp/LocalSettings.php .
# vi LocalSettings.php
--update $wgDBpassword
--update $wgServer

# cd /var/www
# chown -R apache:apache html
# cd ~david
--couldn't deal with the collation
# sed -i 's/utf8mb4_0900_ai_ci/utf8mb4_unicode_ci/g' my_wiki.sql 
# mysql -p < my_wiki.sql my_wiki
# systemctl restart mariadb
# systemctl restart httpd


### enabling uploads in mediawiki
```
# cp -p /etc/php.ini /etc/php.ini-2025-08-13
# vi /etc/php.ini
upload_tmp_dir = /var/www/html/upload-tmp-dir 
# mkdir /var/www/html/upload-tmp-dir ; chown apache:apache /var/www/html/upload-tmp-dir
# chown apache:apache /var/www/html/mediawiki/images/ 
# systemctl restart httpd
# chcon -R -t httpd_sys_rw_content_t /var/www/html/mediawiki-1.44.0/images

--removing some of the noise from /var/log/messages
# setsebool -P httpd_can_network_connect 1
# setsebool -P httpd_graceful_shutdown 1

--keep trying
# semanage fcontext -a -t httpd_tmp_t "/var/www/html/upload-tmp-dir(/.*)?"

root@oralinux01:/var/www/html/mediawiki# ls -Z includes/GlobalFunctions.php
unconfined_u:object_r:httpd_sys_content_t:s0 includes/GlobalFunctions.php
root@oralinux01:/var/www/html/mediawiki# chcon -t httpd_sys_script_exec_t includes/GlobalFunctions.php
# semanage fcontext -a -t httpd_sys_script_exec_t /var/www/html/mediawiki-1.44.0/includes/GlobalFunctions.php

# chcon -R -t httpd_user_rw_content_t images
# semanage fcontext -a -t httpd_user_rw_content_t "/var/www/html/mediawiki-1.44.0/images(/.*)?"
# restorecon -vR /var/www/html/mediawiki-1.44.0/
```

### selinux basics
```
root@oralinux01:/var/www/html# sestatus

SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

# ps axZ
# netstat -tnlpZ
# semanage port -l
# id -Z

--trying to catch all the errors.
change SELinux to permissive
# setenforce 0

then do the app upload file..

```
