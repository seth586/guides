[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ]  - [ **nginx** ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install nginx
```
# pkg install nginx redis
# sysrc nginx_enable=yes
# sysrc redis_enable=yes
# nano /usr/local/etc/nginx.conf
```

```
# TEST CONFIG ONLY, DO NOT USE IN PRODUCTION
events{}
http {

    server {
        listen       80;
        root   /usr/local/www/nginx;
        index  index.php index.html index.htm;

        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass   unix:/var/run/php-fpm.sock;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME $request_filename;
            include        fastcgi_params;
        }
    }
}

```
Save (CTRL+O, ENTER) and exit (CTRL+X)

Start the service
```
# service nginx start
```

### Test nginx
navigate to `your.jail.ip.address` in a browser, you should see the nginx welcome page.

### Configure redis
`nano /usr/local/etc/redis.conf`:
```
port 0
unixsocket /var/run/redis/redis.sock
unixsocketperm 770
bind 127.0.0.1
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

