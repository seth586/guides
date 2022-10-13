[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [nginx](4_apache.md) ] - [ **PHP** ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install prerequisites:
```
# pkg install php82 php82-ctype php82-curl php82-dom php82-filter php82-gd php82-mbstring php82-posix php82-session php82-xml php82-simplexml php82-xmlreader php82-xmlwriter php82-zip php82-zlib php82-pdo_mysql php82-fileinfo php82-bz2 php82-intl php82-bcmath php82-gmp php82-exif php82-pecl-redis php82-pecl-imagick php82-pcntl php82-phar php82-pecl-redis
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

post_max_size = 1999M
upload_max_filesize = 1999M
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
```

### Test your php installation
`nano /usr/local/www/nginx/info.php`:
```
<?php
phpinfo(); //display all info
?>
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

Naigate to `your.jail.ip.address/info.php`

Remove once you confirmed it works `rm /usr/local/www/nginx/info.php`
