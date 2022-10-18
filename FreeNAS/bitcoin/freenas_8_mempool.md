[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor & i2p](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - **[mempool]** - [ [Extras](extras.md) ]

## TrueNASnode - full bitcoin stack deployment guide ![BSDBTC60.png](images/BSDBTC60.png)

Join the chatroom on the matrix chat protocol: [#truenasnode:nym.im](https://matrix.to/#/#truenasnode:nym.im)

## mempool

When transacting on chain, you need a good understanding on what network fees are currently trending so you can set your transaction priority accordingly. The best visual tool I've ever found is this gem called [mempool](https://github.com/mempool/mempool). 

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
We are going to jail this process seperately in case you want to expose this website to clearnet. Create a new jail with the process described [in the beginning of this guide](freenas_1_jail_creation.md). Name it mempool or the like. Enter the jail.
```
root@freenas[~]# iocage console mempool
root@mempool:~ #
```

Lets get installing!
```
# pkg update && pkg upgrade -y && pkg install -y nano ca_root_nss node16 npm-node16 nginx mariadb105-server mariadb105-client
# fetch https://github.com/mempool/mempool/archive/refs/tags/v2.3.1.tar.gz
# tar -xvf v2.3.1.tar.gz
# rm v2.3.1.tar.gz
# mv ~/mempool-2.3.1 ~/mempool
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
> create database mempool;
> grant all privileges on mempool.* to 'mempool' identified by 'password123';
> FLUSH PRIVILEGES;
> exit
```

### Setup the web server nginx
```
# sysrc nginx_enable="YES"
# rm /usr/local/etc/nginx/nginx.conf
# nano /usr/local/etc/nginx/nginx.conf
```

Populate `nginx.conf` with the following:
```
user nobody;
pid /var/run/nginx.pid;

worker_processes auto;
worker_rlimit_nofile 100000;

events {
        worker_connections 9000;
        multi_accept on;
}

http {
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;

        server_tokens off;
        server_name_in_redirect off;

        include /usr/local/etc/nginx/mime.types;
        default_type application/octet-stream;

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        # reset timed out connections freeing ram
        reset_timedout_connection on;
        # maximum time between packets the client can pause when sending nginx any data
        client_body_timeout 10s;
        # maximum time the client has to send the entire header to nginx
        client_header_timeout 10s;
        # timeout which a single keep-alive client connection will stay open
        keepalive_timeout 69s;
        # maximum time between packets nginx is allowed to pause when sending the client data
        send_timeout 10s;

        # number of requests per connection, does not affect SPDY
        keepalive_requests 100;

        # enable gzip compression
        gzip on;
        gzip_vary on;
        gzip_comp_level 6;
        gzip_min_length 1000;
        gzip_proxied expired no-cache no-store private auth;
        # text/html is always compressed by gzip module
        gzip_types application/javascript application/json application/ld+json application/manifest+json application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard;

        # limit request body size
        client_max_body_size 10m;

        # proxy cache
        proxy_cache off;
        proxy_cache_path /var/cache/nginx keys_zone=cache:20m levels=1:2 inactive=600s max_size=500m;
        types_hash_max_size 2048;

        # exempt localhost from rate limit
        geo $limited_ip {
                default         1;
                127.0.0.1       0;
        }
        map $limited_ip $limited_ip_key {
                1 $binary_remote_addr;
                0 '';
        }

        # rate limit requests
        limit_req_zone $limited_ip_key zone=api:5m rate=200r/m;
        limit_req_zone $limited_ip_key zone=electrs:5m rate=2000r/m;
        limit_req_status 429;

        # rate limit connections
        limit_conn_zone $limited_ip_key zone=websocket:10m;
        limit_conn_status 429;

        map $http_accept_language $header_lang {
                default en-US;
                ~*^en-US en-US;
                ~*^en en-US;
                ~*^ar ar;
                ~*^cs cs;
                ~*^de de;
                ~*^es es;
                ~*^fa fa;
                ~*^fr fr;
                ~*^ko ko;
                ~*^it it;
                ~*^he he;
                ~*^ka ka;
                ~*^hu hu;
                ~*^nl nl;
                ~*^ja ja;
                ~*^nb nb;
                ~*^pl pl;
                ~*^pt pt;
                ~*^sl sl;
                ~*^fi fi;
                ~*^sv sv;
                ~*^tr tr;
                ~*^uk uk;
                ~*^vi vi;
                ~*^zh zh;
                ~*^ru ru;
        }

        map $cookie_lang $lang {
                default $header_lang;
                ~*^en-US en-US;
                ~*^en en-US;
                ~*^ar ar;
                ~*^cs cs;
                ~*^de de;
                ~*^es es;
                ~*^fa fa;
                ~*^fr fr;
                ~*^ko ko;
                ~*^it it;
                ~*^he he;
                ~*^ka ka;
                ~*^hu hu;
                ~*^nl nl;
                ~*^ja ja;
                ~*^nb nb;
                ~*^pl pl;
                ~*^pt pt;
                ~*^sl sl;
                ~*^fi fi;
                ~*^sv sv;
                ~*^tr tr;
                ~*^uk uk;
                ~*^vi vi;
                ~*^zh zh;
                ~*^ru ru;
        }

        server {
                listen 80;
                include /usr/local/etc/nginx/nginx-mempool.conf;
        }
}
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

Edit nginx-mempool.conf: `nano /usr/local/etc/nginx/nginx-mempool.conf`:
```
root /usr/local/www/mempool/browser;

        index index.html;

        # fallback for all URLs i.e. /address/foo /tx/foo /block/000
        location / {
                add_header Vary Cookie;
                try_files /$lang/$uri /$lang/$uri/ $uri $uri/ /en-US/$uri @index-redirect;
        }
        location @index-redirect {
                add_header Vary Cookie;
                rewrite (.*) /$lang/index.html;
        }

        # location block using regex are matched in order

        # used to rewrite resources from /<lang>/ to /en-US/
        location ~ ^/(ar|bg|bs|ca|cs|da|de|et|el|es|eo|eu|fa|fr|gl|ko|hr|id|it|he|ka|lv|lt|hu|mk|ms|nl|ja|ka|no|nb|nn|pl|pt|pt-BR|ro|ru|sk|sl|sr|sh|fi|sv|th|tr|uk|> ) {
                add_header Vary Cookie;
                rewrite ^/[a-zA-Z-]*/resources/(.*) /en-US/resources/$1;
        }
        # used for cookie override
        location ~ ^/(ar|bg|bs|ca|cs|da|de|et|el|es|eo|eu|fa|fr|gl|ko|hr|id|it|he|ka|lv|lt|hu|mk|ms|nl|ja|ka|no|nb|nn|pl|pt|pt-BR|ro|ru|sk|sl|sr|sh|fi|sv|th|tr|uk|> ) {
                add_header Vary Cookie;
                try_files $uri $uri/ /$1/index.html =404;
        }

        # static API docs
        location = /api {
                try_files $uri $uri/ /en-US/index.html =404;
        }
        location = /api/ {
                try_files $uri $uri/ /en-US/index.html =404;
        }

        # mainnet API
        location /api/v1/donations {
                proxy_pass https://mempool.space;
        }
        location /api/v1/donations/images {
                proxy_pass https://mempool.space;
        }
        location /api/v1/contributors {
                proxy_pass https://mempool.space;
        }
        location /api/v1/contributors/images {
                proxy_pass https://mempool.space;
        }
        location /api/v1/ws {
                proxy_pass http://127.0.0.1:8999/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
        }
        location /api/v1 {
                proxy_pass http://127.0.0.1:8999/api/v1;
        }
        location /api/ {
                proxy_pass http://127.0.0.1:8999/api/v1/;
        }

        # mainnet API
        location /ws {
                proxy_pass http://127.0.0.1:8999/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
        }
```

```
# service nginx start
```

### Compile and deploy

```
# cd ~/mempool/backend
# npm install --prod
# npm run build
# cp mempool-config.sample.json mempool-config.json
# nano mempool-config.json
```

Configure mempool backend to connect to bitcoin core:
```
{
  "MEMPOOL": {
    "NETWORK": "mainnet",
    "BACKEND": "electrum",
    "HTTP_PORT": 8999,
    "SPAWN_CLUSTER_PROCS": 0,
    "API_URL_PREFIX": "/api/v1/",
    "POLL_RATE_MS": 2000,
    "CACHE_DIR": "./cache",
    "CLEAR_PROTECTION_MINUTES": 20,
    "RECOMMENDED_FEE_PERCENTILE": 50
  },
  "CORE_RPC": {
    "HOST": "192.168.84.21",
    "PORT": 8332,
    "USERNAME": "mempool",
    "PASSWORD": "2tm5NiN8wZVyjx_hgUL5O8it68WfoadHDEZ-v6w_RhQ="
  },
  "ELECTRUM": {
    "HOST": "192.168.84.21",
    "PORT": 50001,
    "TLS_ENABLED": false
  },
  "ESPLORA": {
    "REST_API_URL": "http://127.0.0.1:3000"
  },
  "CORE_RPC_MINFEE": {
    "ENABLED": false,
    "HOST": "127.0.0.1",
    "PORT": 8332,
    "USERNAME": "mempool",
    "PASSWORD": "mempool"
  },
  "DATABASE": {
    "ENABLED": true,
    "HOST": "127.0.0.1",
    "PORT": 3306,
    "DATABASE": "mempool",
    "USERNAME": "mempool",
    "PASSWORD": "password123"
  },
  "SYSLOG": {
    "ENABLED": false,
    "HOST": "127.0.0.1",
    "PORT": 514,
    "MIN_PRIORITY": "info",
    "FACILITY": "local7"
  },
  "STATISTICS": {
    "ENABLED": false,
    "TX_PER_SECOND_SAMPLE_PERIOD": 150
  },
  "BISQ": {
    "ENABLED": false,
    "DATA_PATH": "/bisq/statsnode-data/btc_mainnet/db"
  }
}
```
Save (CTRL+O,ENTER) and exit (CTRL+X). Save a backup of your config to `/usr/local/etc/mempool-config.json`:
```
# cp mempool-config.json /usr/local/etc/mempool-config.json
```

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
# REQUIRE: mysql
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
# npm install
# npm run build
# rsync -av --delete dist/mempool /usr/local/www/
```

Navigate to mempool's jail IP and you should have a working website!

## Upgrade Mempool
```
# service mempoolbackend stop && service nginx stop
# cd ~
# fetch https://github.com/mempool/mempool/archive/refs/tags/v2.2.1.tar.gz
# tar -xvf v2.2.1.tar.gz && rm v2.2.1.tar.gz
# rm ~/mempool && mv ~/mempool-2.2.1 ~/mempool
# cd ~/mempool/backend
# touch ~/mempool/backend/cache.json
# npm install
# npm run build
# cp /usr/local/etc/mempool-config.json ~/mempool/backend/mempool-config.json
# cd ~/mempool/frontend
# npm install
# npm run build
# rsync -av --delete dist/mempool /usr/local/www/
# service mempoolbackend start && service nginx start
```

Next: [ [Extras](extras.md) ]
