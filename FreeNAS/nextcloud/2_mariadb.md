[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ **mariadb** ] - [ [PHP](3_php.md) ] - [ [nginx](4_apache.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install 

```
# pkg install nano wget ca_root_nss mariadb106-server
# sysrc mysql_enable=yes
# service mysql-server start
```

### Setup
```
# mysql_secure_installation
Enter current password for root (enter for none):
Switch to unix_socket authentication [Y/n] y
Set root password? [Y/n] y
New password: 
Re-enter new password: 
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

### Create Database
```
# mysql -u root -p
CREATE DATABASE nextcloud;
CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'your-password-here';
GRANT ALL ON nextcloud.* TO 'nextcloud'@'localhost';
FLUSH PRIVILEGES;
exit
```

### Required configuration parameters 
See [here](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html) for official documentation required config paramters:

`nano /usr/local/etc/mysql/conf.d/server.cnf`:
```
[mysqld]
...
transaction_isolation = READ-COMMITTED
binlog_format = ROW
...
```
`nano /usr/local/etc/php/ext-30-pdo_mysql.ini`:
```
extension=pdo_mysql.so

[mysql]
mysql.default_socket=/var/run/mysql/mysql.sock
```


