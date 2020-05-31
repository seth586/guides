[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - **[mempool]** - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

## mempool

When transacting on chain, you need a good understanding on what network fees are currently trending so you can set yuour transaction priority accordingly. The best visual tool I've ever found is this gem called [mempool](https://github.com/mempool/mempool). 

## Create RPC credentials for Bitcoin Core

SSH into your bitcoin jail.

```
# iocage console bitcoin
```

Download the rpcauth tool as documented [here](https://github.com/bitcoin/bitcoin/tree/master/share/rpcauth).

```
# fetch https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py
# python3 ./rpcauth.py mempool
String to be appended to bitcoin.conf:
rpcauth=mempool:5d0d70936350d0a79b588a9bb2906ea1$82afc2d29dfcfd808acd98f855cf47989564d8f1cd55b515f23fb10ace0dd75a
Your password:
2tm5NiN8wZVyjx_hgUL5O8it68WfoadHDEZ-v6w_RhQ=
```

Save this information. Add the `rpcauth=` string above to `bitcoin.conf` and configure rpc access:
```
# nano /usr/local/etc/bitcoin.conf
rpcauth=mempool:5d0d70936350d0a79b588a9bb2906ea1$82afc2d29dfcfd808acd98f855cf47989564d8f1cd55b515f23fb10ace0dd75a
rpcallowip=192.168.84.0/24
rpcbind=0.0.0.0
```
Make sure that the `rcpallowip=` coorelates to your local subnet address range.

Save (CTRL+O,ENTER) and exit (CTRL+X)

Reboot bitcoin core, make sure bitcoind is running sucessfuly after the reboot by running `ps aux`.
```
# service bitcoind restart
# ps aux
USER      PID %CPU %MEM     VSZ     RSS TT  STAT STARTED      TIME COMMAND
bitcoin 76206  4.4  2.2 4551716 1449288  -  SJ   Wed23   426:49.66 /usr/local/bin/bitcoind -conf=/usr/local/etc/bitcoin.conf -datadir=/var/db/bitcoin
...
```
We are done in your bitcoin jail, so exit the jail:
```
# exit
logout
root@freenas[~]#
```

Create a new jail with the process described [in the beginning of this guide](freenas_1_jail_creation.md). Name it mempool or the like. Enter the jail.
```
root@freenas[~]# iocage console mempool
root@mempool:~ #
```

Lets get installing!
```
pkg update && pkg upgrade -y && pkg install nano ca_root_nss npm-node12 nginx mariadb104-server mariadb104-client git
```

Setup our MariaDB database:
```
# sysrc mysql_enable="YES"
# service mysql-server onestart
# mysql_secure_installation
# mysql -u root -p
> create database mempool;
> grant all privileges on mempool.* to 'mempool' identified by 'password123';
> FLUSH PRIVILEGES;
> exit
# mysql -u mempool -p password123 < /root/mempool/mariadb-structure.sql
```

Setup the web server nginx
```
# sysrc nginx_enable="YES"
# service nginx start
```

open browser to jail IP, you should see "welcome to nginx!" message

```
# service nginx stop
# mkdir /usr/local/www/mempool.lan
# chown -R www:www /usr/local/www/mempool.lan
# nano /usr/local/etc/nginx/nginx.conf
```

Populate `nginx.conf` with the following:
```
user www;
worker_processes 1;
pid /var/run/nginx.pid;
#include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;

	include /usr/local/etc/nginx/mime.types;
	default_type application/octet-stream;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	gzip on;
	gzip_comp_level    5;
	gzip_min_length    256;
	gzip_proxied       any;
	gzip_vary          on;

	gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy; # text/html is always compressed by gzip module

	server {
		listen 80;
		listen [::]:80;
		server_name mempool.lan;

		root /usr/local/www/mempool.lan;

		index index.html;
		server_name mempool.lan; # managed by Certbot

		location / {
			try_files $uri $uri/ /index.html =404;
		}

		location /api {
			proxy_pass http://127.0.0.1:8999/api;
		}

		location /ws {
			proxy_pass http://127.0.0.1:8999/;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "Upgrade";
		}
	}
}
```
Save (CTRL+O, ENTER) and exit (CTRL+X)
