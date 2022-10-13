[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ]  - [ **nginx** ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install nginx
```
# pkg install nginx
# sysrc nginx_enable=yes
# nano /usr/local/etc/nginx.conf
```

```
user  www;
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    client_max_body_size 8M;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;
        root   /usr/local/www/nginx;
        index  index.php index.html index.htm;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/local/www/nginx-dist;
        }

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

