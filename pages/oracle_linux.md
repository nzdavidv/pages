## Oracle Linux lab

10 mins in and massively impressed with oracle Linux 10.

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

root@oralinux01:/var/www/html# mysql_secure_installation

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
# gunzip mediawiki-1.44.0.tar.gz  
# tar xvf mediawiki-1.44.0.tar
# chown -R apache:apache mediawiki-1.44.0/
# ln -s mediawiki-1.44.0/ mediawiki
```

### restoring from raspidev
copied backup files over to ~www/restore 
```
# cd /var/www
# tar xvf ~www/restore/var-www-html-raspidev.tar 
# cd /var/www/html/mediawiki-1.43.0/images/
# mv * ../../mediawiki-1.44.0/images/
# cd /var/www
# tar xvf ~www/restore/var-www-html-raspidev.tar 

# cd /var/www/html/
# vi mediawiki-1.44.0/LocalSettings.php
$wgEnableUploads = true;

# cd /var/www
# chown -R apache:apache html
# cd ~www/restore/
--couldn't deal with the collation
# sed -i 's/utf8mb4_0900_ai_ci/utf8mb4_unicode_ci/g' my_wiki-raspidev.sql 
# mysql -p < my_wiki-raspidev.sql my_wiki

--the restore overwrote my symlink
# cd /var/www/html
# rm mediawiki
rm: remove symbolic link 'mediawiki'? y
# ln -s mediawiki-1.44.0/ mediawiki
# mv mediawiki-1.43.0/ old_medwiki
# systemctl restart httpd
# systemctl restart mariadb

```

```
chown -R apache:apache *
   # if you have folder to store uploaded file (optional)
   sudo chcon -R -t httpd_sys_rw_content_t docsuploaded
   find /var/www/html -type d -exec chmod 755 {} \;
   find /var/www/html -type f -exec chmod 644 {} \;
   systemctl restart httpd
```
