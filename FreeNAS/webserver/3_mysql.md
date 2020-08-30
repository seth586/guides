[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

## Guide to a self hosted wordpress website on FreeNAS/TrueNAS ![wordpress60.png](images/wordpress60.png)
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - **[mysql]** - [ [PHP](4_php.md) ] - [ [wordpress](5_wordpress.md) ] - [ [reverse proxy](6_reverse_proxy.md) ]

MySQL is a database structure for storing and recalling information. When you use wordpress to make a blog post, or allow users to comment, or create a transaction, this data is stored and retreived from the database. There are several MySQL clones out there, and my personal favorite is MariaDB.

## Install MariaDB
```
# pkg search mariadb
# pkg install -y mariadb104-server
# sysrc mysql_enable=yes
# service mysql-server start
# ps aux
```
Verify thhat mysql is running.
```
mysql 56864  0.0  0.0   7068  2776  -  IsJ  20:32   0:00.01 /bin/sh /usr/local/bin/mysqld_safe --defaults-extra-file=/var/db/mysql/my.cnf --user=mysql --datadir=/var/db/mysql --pid-file=/var/db/mysql/blog.pid
mysql 56933  0.0  0.1 583512 91640  -  IJ   20:32   0:00.15 /usr/local/libexec/mysqld --defaults-extra-file=/var/db/mysql/my.cnf --basedir=/usr/local --datadir=/var/db/mysql --plugin-dir=/usr/local/lib/mysql/plugin --log-error=/var/db/mysql/blog.err --pid-file=/var/db/mysql/blog.pid
```

## Secure the database installation
```
$ mysql_secure_installation
Enter current password for root (enter for none):
Switch to unix_socket authentication [Y/n] Y
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

## Create your website database 
Press ENTER since there is no root password required. (It is secured by SSH login) Replace `database_name_here`, `username_here`, and `password_here`. Do not lose this information!
```
# mysql -u root -p
Enter password:
> CREATE DATABASE database_name_here;
> GRANT ALL PRIVILEGES ON database_name_here.* TO 'username_here'@'localhost' IDENTIFIED BY 'password_here';
> FLUSH PRIVILEGES;
> exit
```

## Upgrade mariadb
Make sure to back up your database prior to an upgrade attempt!
```
# pkg update
# pkg upgrade mariadb104-server
# service mariadb-server restart
# pkg upgrade mariadb104
# mariadb-upgrade --datadir=/var/db/mysql
```

Next: [ [PHP](4_php.md) ] >>
