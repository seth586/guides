
[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [postgresql](2_postgresql.md) ] - [ **synapse** ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Synapse

Official & up to date install instructions are maintained [here](https://matrix-org.github.io/synapse/latest/setup/installation.html)

### Switch jails and set permissions for mount points:
```
# exit
root@truenas[~]# iocage console synapse
# pw adduser synapse -d /nonexistent -s /usr/sbin/nologin
# pw usermod synapse -G wheel
# chown -R synapse /usr/local/etc/matrix-synapse
# chown -R synapse /var/db/matrix-synapse
```

### Install
```
pkg install rust nano py39-virtualenv py39-pip py39-pillow postgresql16-client


mkdir -p ~/synapse
virtualenv -p python3 ~/synapse/env
source ~/synapse/env/bin/activate.csh
pip install --upgrade pip
pip install --upgrade setuptools
pip index versions matrix-synapse
pip install "matrix-synapse[postgres]"==1.95.0
```
### Create config
Full config instructions are maintained [here](https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html)
```
# cd ~/synapse
# python3.9 -m synapse.app.homeserver \
    --server-name mydomain.com \
    --config-path /usr/local/etc/matrix-synapse/homeserver.yaml \
    --generate-config \
    --data-directory /var/db/matrix-synapse \
    --report-stats=no
```
`nano /usr/local/etc/matrix-synapse/homeserver.yaml`:
```
pid_file: /var/run/matrix-synapse/homeserver.pid

    bind_addresses: ['::1', '192.168.84.79']

database:
  name: psycopg2
  txn_limit: 10000
  args:
    user: synapse
    password: password
    database: synapse
    host: 192.168.84.78
    port: 5432
    cp_min: 5
    cp_max: 10
```


`nano /usr/local/etc/matrix-synapse/mydomain.com.log.config`:
```
        filename: /var/log/matrix-synapse/homeserver.log
```


### Autostart
```
# mkdir /usr/local/etc/rc.d && touch /usr/local/etc/rc.d/synapse && chmod +x /usr/local/etc/rc.d/synapse && nano /usr/local/etc/rc.d/synapse
```

```
#!/bin/sh
#
# Created by: Mark Felder <feld@FreeBSD.org>

# PROVIDE: synapse
# REQUIRE: LOGIN postgresql
# KEYWORD: shutdown

#
# Add the following line to /etc/rc.conf to enable `synapse':
#
# synapse_enable="YES"

. /etc/rc.subr
name=synapse

rcvar=synapse_enable
load_rc_config ${name}

: ${synapse_enable:=NO}
: ${synapse_user:=synapse}
: ${synapse_conf:=/usr/local/etc/matrix-synapse/homeserver.yaml}
: ${synapse_dbdir:=/var/db/matrix-synapse}
: ${synapse_logdir:=/var/log/matrix-synapse}
: ${synapse_pidfile:=/var/run/matrix-synapse/homeserver.pid}

pidfile="${synapse_pidfile}"
procname=/root/synapse/env/bin/python3.9
command=/root/synapse/env/bin/python3.9
command_args="-m synapse.app.homeserver --daemonize -c ${synapse_conf}"
start_precmd=start_precmd

start_precmd()
{
        if [ ! -d ${synapse_pidfile%/*} ] ; then
                install -d -o ${synapse_user} -g wheel ${synapse_pidfile%/*};
        fi

        if [ ! -d ${synapse_dbdir} ] ; then
                install -d -o ${synapse_user} -g wheel ${synapse_dbdir};
        fi

        if [ ! -d ${synapse_logdir} ] ; then
                install -d -o ${synapse_user} -g wheel ${synapse_logdir};
        fi
}

run_rc_command "$1"
```
```
sysrc synapse_enable="YES"
```




### start the service 
```
# service synapse start && tail -f /var/log/matrix-synapse/homeserver.log
```
### To Upgrade:
```
# service synapse stop
# source ~/synapse/env/bin/activate
# pip install --upgrade pip
# pip install --upgrade setuptools 
# pip install -U "matrix-synapse[postgres]"==1.95.1
# exit
# service synapse start
```    
