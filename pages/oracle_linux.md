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
# syste#mctl restart httpd
--config firewall
# firewall-cmd --add-service=http --permanent
# firewall-cmd --reload

-- install php mysql module
# dnf install php-mysqli
```

### database

```
# systemctl start mariadb

root@oralinux01:/var/www/html# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

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


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

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


```

```
chown -R apache:apache *
   # if you have folder to store uploaded file (optional)
   sudo chcon -R -t httpd_sys_rw_content_t docsuploaded
   find /var/www/html -type d -exec chmod 755 {} \;
   find /var/www/html -type f -exec chmod 644 {} \;
   systemctl restart httpd
```
