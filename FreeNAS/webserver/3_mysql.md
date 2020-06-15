## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - **[mysql]** - [ [PHP](4_php.md) ] - [ [wordpress](5_wordpress.md) ] - [ [reverse proxy](6_reverse_proxy.md) ]

MySQL is a database structure for storing and recalling information. When you use wordpress to make a blog post, or allow users to comment, or create a transaction, this data is stored and retreived from the database. There are several MySQL clones out there, and my personal favorite is MariaDB.

## Install MariaDB
```
$ pkg search mariadb
$ pkg install -y mariadb104-server
$ sysrc mysql_enable=yes
$ service mysql-server start
```

## Configure the database installation
```
$ mysql_secure_installation
```

## Create your website database 
Replace `database_name_here`, `username_here`, and `password_here`. Do not lose this information!
```
# mysql -u root -p
> CREATE DATABASE database_name_here;
> GRANT ALL PRIVILEGES ON database_name_here.* TO 'username_here'@'localhost' IDENTIFIED BY 'password_here';
> FLUSH PRIVILEGES;
> exit
```

Next: [ [PHP](4_php.md) ] >>
