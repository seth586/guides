[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - **[mempool]** - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

## mempool

When transacting on chain, you need a good understanding on what network fees are currently trending so you can set yuour transaction priority accordingly. The best visual tool I've ever found is this gem called [mempool](https://github.com/mempool/mempool). 

### Create RPC credentials for Bitcoin Core

SSH into your bitcoin jail.

```
# iocage console bitcoin
```

Download the rpcauth tool as documented [here](https://github.com/bitcoin/bitcoin/tree/master/share/rpcauth). Save this information.

```
# fetch https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py
# python3 ./rpcauth.py mempool
String to be appended to bitcoin.conf:
rpcauth=mempool:5d0d70936350d0a79b588a9bb2906ea1$82afc2d29dfcfd808acd98f855cf47989564d8f1cd55b515f23fb10ace0dd75a
Your password:
2tm5NiN8wZVyjx_hgUL5O8it68WfoadHDEZ-v6w_RhQ=
```

Add the `rpcauth=` string above to `bitcoin.conf` and configure rpc access. Make sure that the `rcpallowip=` coorelates to your local subnet address range.
```
# nano /usr/local/etc/bitcoin.conf
rpcauth=mempool:5d0d70936350d0a79b588a9bb2906ea1$82afc2d29dfcfd808acd98f855cf47989564d8f1cd55b515f23fb10ace0dd75a
rpcallowip=192.168.84.0/24
rpcbind=0.0.0.0
```
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
### Create new jail for mempool
Create a new jail with the process described [in the beginning of this guide](freenas_1_jail_creation.md). Name it mempool or the like. Enter the jail.
```
root@freenas[~]# iocage console mempool
root@mempool:~ #
```

Lets get installing!
```
# pkg update && pkg upgrade -y && pkg install -y nano ca_root_nss npm-node12 nginx mariadb104-server mariadb104-client git
# fetch https://github.com/mempool/mempool/archive/v1.0.0.tar.gz
# tar -xvf v1.0.0.tar.gz
# rm v1.0.0.tar.gz
# mv ~/mempool-1.0.0 ~/mempool
```

### Setup MariaDB database:
```
# sysrc mysql_enable="YES"
# service mysql-server onestart
# mysql_secure_installation
```
Follow the prompts to secure your database:
```
Enter current password for root (enter for none): [ENTER]
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

```
# mysql -u root -p
Enter password: [ENTER]
> create database mempooldb;
> grant all privileges on mempooldb.* to 'mempooluser' identified by 'password123';
> FLUSH PRIVILEGES;
> exit
# mysql -u mempooluser -p mempooldb < /root/mempool/mariadb-structure.sql
Enter password: password123
#
```

Setup the web server nginx
```
# sysrc nginx_enable="YES"
# service nginx start
```

open browser to jail IP, you should see "welcome to nginx!" message

```
# service nginx stop
# rm /usr/local/etc/nginx/nginx.conf
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

		root /usr/local/www;

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

```
# service nginx start
```

### Compile and deploy

```
# cd ~/mempool/backend
# npm install -g typescript
# touch ~/mempool/backend/cache.json
# npm install
# cp mempool-config.sample.json mempool-config.json
# nano mempool-config.json
```

Configure mempool backend to connect to bitcoin core:
```
{
  "ENV": "dev",
  "DB_HOST": "localhost",
  "DB_PORT": 3306,
  "DB_USER": "mempooluser",
  "DB_PASSWORD": "password123",
  "DB_DATABASE": "mempooldb",
  "HTTP_PORT": 3000,
  "API_ENDPOINT": "/api/v1/",
  "CHAT_SSL_ENABLED": false,
  "CHAT_SSL_PRIVKEY": "",
  "CHAT_SSL_CERT": "",
  "CHAT_SSL_CHAIN": "",
  "MEMPOOL_REFRESH_RATE_MS": 500,
  "INITIAL_BLOCK_AMOUNT": 8,
  "DEFAULT_PROJECTED_BLOCKS_AMOUNT": 3,
  "KEEP_BLOCK_AMOUNT": 24,
  "BITCOIN_NODE_HOST": "192.168.84.208",
  "BITCOIN_NODE_PORT": 8332,
  "BITCOIN_NODE_USER": "mempool",
  "BITCOIN_NODE_PASS": "2tm5NiN8wZVyjx_hgUL5O8it68WfoadHDEZ-v6w_RhQ=",
  "BACKEND_API": "bitcoind",
  "ELECTRS_API_URL": "https://www.blockstream.info/api",
  "TX_PER_SECOND_SPAN_SECONDS": 150
}

```
Save (CTRL+O,ENTER) and exit (CTRL+X)

Now lets give it a test run. If everything is properly configured, you should see something like the following:

```
root@mempool:~/mempool/backend # npm run start

> mempool-backend@1.0.0 start /root/mempool/backend
> npm run build && node dist/index.js


> mempool-backend@1.0.0 build /root/mempool/backend
> tsc

Server started on 127.0.0.1:8999
Statistics cache created
Fetching block tx 1 of 1492
Fetching block tx 2 of 1492
Fetching block tx 3 of 1492
...
```
Press CTRL+C to abort the process.

### Configure backend as a service and autostart

```
# nano /usr/local/etc/rc.d/mempoolbackend
```

Paste the following:

```
#!/bin/sh
#
# PROVIDE: mempoolbackend
# REQUIRE: 
# KEYWORD:

. /etc/rc.subr
name="mempoolbackend"
rcvar="mempoolbackend_enable"
mempoolbackend_chdir="/root/mempool/backend"
mempoolbackend_command="/usr/local/bin/node /root/mempool/backend/dist"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${mempoolbackend_command}"

load_rc_config $name
: ${mempoolbackend_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

```
# chmod +x /usr/local/etc/rc.d/mempoolbackend
# sysrc mempoolbackend_enable=YES
# service mempoolbackend start
```

### Install frontend
```
# cd ~/mempool/frontend
# npm i @angular-devkit/build-angular@0.803.24
# npm run build
# rsync -av --delete dist/mempool/ /usr/local/www/
```

Navigate to mempool's jail IP and you should have a working website!

Next: [ [Extras](extras.md) ]
