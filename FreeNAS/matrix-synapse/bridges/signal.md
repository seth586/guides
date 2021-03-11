## Install signald

???

## Create new database
```
# nano /var/lib/postgres/data13/pg_hba.conf

host    mautrix-signal  mautrix-signal  127.0.0.1/32         password
```

```
root@synapse:~ # sudo -i -u postgres
$ psql
postgres=# CREATE USER "mautrix-signal" WITH PASSWORD 'password';
postgres=# CREATE DATABASE "mautrix-signal" OWNER "mautrix-signal";
postgres=# \q
$ pg_ctl reload -D /var/db/postgres/data13
$ exit
root@synapse:~ #
```

## Install mautrix-signal
```
pkg install py37-virtualenv olm rust py37-pillow nano

pw adduser mautrix-signal -d /nonexistent -s /usr/sbin/nologin -c "User for mautrix-signal bridge"

mkdir /var/db/mautrix-signal && chown -R mautrix-signal:mautrix-signal /var/db/mautrix-signal

cd /var/db/mautrix-signal

virtualenv -p /usr/local/bin/python3.7 .

source /var/db/mautrix-signal/bin/activate.csh

pip install --global-option=build_ext --global-option="-I/usr/local/include" --upgrade python-olm

pip install --upgrade 'mautrix-signal[all]'

cp example-config.yaml config.yaml
```

Configure mautrix-signal: `nano config.yaml`:
```
homeserver:
    address: https://exmaple.tld
    domain: example.tld
appservice:
    database: postgres://mautrix-signal:password@localhost/mautrix-signal
bridge:
    permissions:
        example.tld: user
        `@admin:example.tld`: admin
logging:
    handlers:
        file:
            filename: /var/db/mautrix-facebook/mautrix-facebook.log
```

Generate `registration.yaml`:
```
# mkdir /usr/local/etc/mautrix-signal
# mv config.yaml /usr/local/etc/mautrix-signal/config.yaml
# sudo -u mautrix-facebook /var/db/mautrix-facebook/bin/python -m mautrix_signal -g -c /usr/local/etc/mautrix-signal/config.yaml -r /usr/local/etc/mautrix-signal/registration.yaml
```
Add mautrix-signal to synapse config `nano ~/.synapse/homeserver.yaml`:
```
app_service_config_files:
  - /usr/local/etc/mautrix-signal/registration.yaml
```
restart synapse:
```
service synapse restart
```
## Create startup script
`touch /usr/local/etc/rc.d/mautrix-signal && chmod +x /usr/local/etc/rc.d/mautrix-signal && nano /usr/local/etc/rc.d/mautrix-signal`:
```
#!/bin/sh
#
# PROVIDE: mautrix_facebook
# REQUIRE:
# KEYWORD:

. /etc/rc.subr
name="mautrix_facebook"
rcvar="mautrix_facebook_enable"
mautrix_facebook_command="/var/db/mautrix-facebook/bin/python -m mautrix_facebook -c /usr/local/etc/mautrix-facebook/config.yaml -r /usr/local/etc/mautrix-facebook/registration.yaml"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u mautrix-facebook -r -f ${mautrix_facebook_command}"

load_rc_config $name
: ${mautrix_facebook_enable:=no}

run_rc_command "$1"
```
## Set permissions
```
# chown -R mautrix-facebook:mautrix-facebook /var/db/mautrix-facebook
# chown -R mautrix-facebook:mautrix-facebook /usr/local/etc/mautrix-facebook
```

## Dry run verbose mode
```
sudo -u mautrix-facebook /var/db/mautrix-facebook/bin/python -m mautrix_signal -c /usr/local/etc/mautrix-signal/config.yaml -r /usr/local/etc/mautrix-signal/registration.yaml
```
(Ctrl+C to stop)
## Start Service
```
# service mautrix-facebook start
# ps aux
mautrix-facebook 10193 50.4  0.1 110020  66508  -  SJ   00:59   0:00.93 /var/db/mautrix-facebook/bin/python -m mautrix_facebook -c /usr/local/etc/mautrix-facebook/config.yaml -r /usr/local/etc/mautrix-facebook/registration.yaml (python3.7)
root             10192  0.0  0.0  10844   2292  -  SsJ  00:59   0:00.00 daemon: /var/db/mautrix-facebook/bin/python[10193] (daemon)
```
