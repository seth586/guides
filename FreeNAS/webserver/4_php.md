[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

## Guide to a self hosted wordpress website on FreeNAS/TrueNAS ![wordpress60.png](images/wordpress60.png)
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - **[PHP]** - [ [wordpress](5_wordpress.md) ] - [ [reverse proxy](6_reverse_proxy.md) ]

PHP is a programming language designed for interactive web content. Numerous PHP modules exist to increase the capability of this language. These PHP modules can be individually installed depending on what your plugins and themes require and isntalled with seperate packages. To see what modules are activated, type `php -m`.

As of writing the latest branch of PHP is version 8.2. Check out this website to see what the latest version is: https://www.php.net/supported-versions.php 

### Install prerequisites:
```
# pkg install php81 php81-mysqli php81-mbstring php81-zlib php81-curl php81-gd php81-exif php81-fileinfo php81-pecl-imagick php81-zip php81-filter php81-iconv php81-xmlwriter php81-opcache php81-simplexml php81-session php81-dom
# sysrc php_fpm_enable=yes

```

### Configure `php.ini`
```
# cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini
# nano /usr/local/etc/php.ini
```

Uncomment and adjust the folllowing:

Note: http://php.net/manual/en/timezones.php for the timezone relevant to you. An example would be Australia/Sydney
```
...
cgi.fix_pathinfo=1
date.timezone=Country/City

post_max_size = 512M
upload_max_filesize = 512M
memory_limit = 512M

opcache.enable=1
opcache.enable_cli=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.revalidate_freq=1
opcache.save_comments=1
...
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

### Change TCP listener to unix socket
`nano /usr/local/etc/php-fpm.d/www.conf`
```
listen = /var/run/php-fpm.sock
listen.owner = www
listen.group = www
listen.mode = 0660
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)
```
# service php-fpm start
# apachectl graceful
```
## Test PHP installation
```
# nano /usr/local/www/nginx/test.php
```
Add the following line:
```
<?php phpinfo(); ?>
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

Navigate to your jail IP on a web browser: ( example `http://192.168.84.80/test.php` ), you should see a website summary of your PHP installation. PHP and nginx are communicating, congradulations! Now lets delete this test file, since leaking your php info can expose you to hackers that may become aware of specific php version vulnerabilities in the future:
 ```
 # rm /usr/local/www/nginx/test.php
 ```
 
 Next: [ [wordpress](5_wordpress.md) ] >>
