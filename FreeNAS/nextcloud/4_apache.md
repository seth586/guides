[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ]  - [ **apache** ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install apache
```
# pkg install apache24 redis nano wget ca_root_nss
# sysrc apache24_enable=yes
# service apache24 start
# sysrc redis_enable=yes 
# nano /usr/local/etc/apache24/httpd.conf
```
Search for and uncomment the following lines (CTRL+W, paste, ENTER)
```
LoadModule proxy_module libexec/apache24/mod_proxy.so
LoadModule proxy_fcgi_module libexec/apache24/mod_proxy_fcgi.so
LoadModule rewrite_module libexec/apache24/mod_rewrite.so
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

Restart the service
```
# apachectl graceful
```

### Test apache & set up test pages
navigate to `your.jail.ip.address` in a browser, you should see the apache "It works!" message.
`nano /usr/local/etc/apache24/Includes/test.conf`: Change server name to jail IP:
```
<VirtualHost *:80>
    DocumentRoot "/usr/local/www/apache24/data"
    ServerName 192.168.0.10
    ProxyPassMatch ^/(.*.php(/.*)?)$ unix:/var/run/php-fpm.sock|fcgi://localhost/usr/local/www/apache24/data/$1
    DirectoryIndex /index.php index.php
</VirtualHost>
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

`nano /usr/local/www/apache24/data/info.php`:
```
<?php
phpinfo(); //display all info
?>
```

### Configure redis
`nano /usr/local/etc/redis.conf`:
```
port 0
unixsocket /var/run/redis/redis.sock
unixsocketperm 770
bind 127.0.0.1
```
Save (CTRL+O, ENTER) and exit (CTRL+X)
```
# pw usermod www -G redis
# service redis start
```

