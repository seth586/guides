[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ]  - [ **nginx** ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install nginx
```
# pkg install nginx
# sysrc nginx_enable=yes
# cp /usr/local/etc/nginx-dist.conf /usr/local/etc/nginx.conf
# nano /usr/local/etc/nginx.conf
```

Uncomment the following section and replace `fastcgi_pass` with a unix socket:
```
location ~ \.php$ {
            root           html;
            fastcgi_pass   unix:/var/run/php-fpm/www.sock;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            include        fastcgi_params;
        }
```


