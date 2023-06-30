[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [apache](4_apache.md) ] - [ [PHP](3_php.md) ] - [ **mariadb** ]  - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install 

```
# pkg install mariadb106-server
# sysrc mysql_enable=yes
# service mysql-server start
```

### Required configuration parameters 
See [here](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html) for official documentation required config paramters:

`nano /usr/local/etc/mysql/conf.d/server.cnf`:
```
[mysqld]
...
character_set_server = utf8mb4
collation_server = utf8mb4_general_ci
transaction_isolation = READ-COMMITTED
binlog_format = ROW
innodb_large_prefix=on
innodb_file_format=barracuda
innodb_file_per_table=1
...
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
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'insert_password_here';
FLUSH PRIVILEGES;
exit
```

### Configure PHP for mysql unix sockets

`nano /usr/local/etc/php/ext-30-pdo_mysql.ini`:
```
extension=pdo_mysql.so

[mysql]
mysql.default_socket=/var/run/mysql/mysql.sock
```
`service php-fpm restart`

### Upgrade MariaDB
Make sure to read the [backup](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/backup.html#mysql-mariadb) and [restoration](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/restore.html) procedure before attempting an upgrade!
```
# service apache24 stop
# cd ~
# mysqldump --single-transaction --default-character-set=utf8mb4 -u nextcloud -pinsert_password_here nextcloud > nextcloud-sqlbkp_`date +"%Y%m%d"`.bak
# service mysql-server stop
# pkg remove mariadb105-server
# pkg install mariadb106-server
# service mysql-server start
# mariadb-upgrade
# service apache24 start
```
