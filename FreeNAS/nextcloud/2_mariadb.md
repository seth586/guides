[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ **mariadb** ] - [ [PHP](3_php.md) ] - [ [apache](4_apache.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

`nano /usr/local/etc/mysql/conf.d/server.cnf`:
```
[mysqld]
innodb_file_per_table           = 1
```
