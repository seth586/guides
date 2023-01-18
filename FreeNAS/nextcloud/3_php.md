[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [apache](4_apache.md) ] - [ **PHP** ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install prerequisites:
```
# pkg install php81 php81-ctype php81-curl php81-dom php81-filter php81-gd php81-mbstring php81-opcache php81-posix php81-session php81-simplexml php81-xmlreader php81-xmlwriter php81-zip php81-zlib php81-pdo_mysql php81-fileinfo php81-bz2 php81-intl php81-bcmath php81-gmp php81-exif php81-pecl-redis php81-pecl-imagick php81-pcntl php81-phar php81-pecl-redis php81-xml
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
# apachectl graceful
```

### Test your php installation
Naigate to `your.jail.ip.address/info.php`

Remove once you confirmed it works `rm /usr/local/www/apache24/data/info.php`
