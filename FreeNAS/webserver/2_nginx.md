[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - **[nginx]** - [ [mysql](3_mysql.md) ] - [ [PHP](4_php.md) ] - [ [wordpress](5_wordpress.md) ] - [ [reverse proxy](6_reverse_proxy.md) ]

nginx (pronounced engine-x) is the web server. It redirects appropriate requests fpr PHP processing if required and then responds to http requests from your users with requested data.

SSH into your new webserver jail.
```
root@TrueNAS[~]# iocage list
+-----+---------------+-------+--------------+---------------+
| JID |     NAME      | STATE |   RELEASE    |      IP4      |
+=====+===============+=======+==============+===============+
| 1   | bitcoin       | up    | 11.3-RELEASE | DHCP          |
+-----+---------------+-------+--------------+---------------+
| 2   | blog          | up    | 11.3-RELEASE | 192.168.84.80 |
+-----+---------------+-------+--------------+---------------+
root@TrueNAS[~]# iocage console blog
...
root@blog:~ #
```
### Install & configure nginx
```
# pkg install -y nano nginx
# sysrc nginx_enable=yes
# rm  /usr/local/etc/nginx/nginx.conf
# nano /usr/local/etc/nginx/nginx.conf
```
Paste the following recomended configuration for a wordpress webserver. Change the `server_name` to the appropriate static ip for the webserver jail.
```
user  www;
worker_processes  1;

#error_log  /var/log/nginx/error.log;

#pid        logs/nginx.pid;


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
        server_name  192.168.84.80;
        root   /usr/local/www/nginx;
        index  index.php index.html index.htm;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/local/www/nginx-dist;
        }

	# This is cool because no php is touched for static content.
        # include the "?$args" part so non-default permalinks doesn't break when using query string
	location / {
            try_files $uri $uri/ /index.php?$args;
        }

        # pass the PHP scripts to FastCGI server listening on unix socket
        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass   unix:/var/run/php-fpm.sock;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME $request_filename;
            include        fastcgi_params;
        }

        # deny access to .htaccess files
        location ~ /\.ht {
            deny  all;
        }
	# cache images
	location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                expires max;
                log_not_found off;
        }
    }
}
```
Save (CTRL+O,ENTER) and exit (CTRL+X)

## Test nginx config file and start service

```
# nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
# service nginx start
```
Open a web browser and navigate to the jail IP address, you should see a basic "welcome to nginx!" website.

Next: [ [mysql](3_mysql.md) ] >>

