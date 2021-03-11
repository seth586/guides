## Create new database
```
# nano /var/db/postgres/data13/pg_hba.conf

host    mautrix-facebook  mautrix-facebook  127.0.0.1/32         password
```

```
root@synapse:~ # sudo -i -u postgres
$ psql
postgres=# CREATE USER "mautrix-facebook" WITH PASSWORD 'password';
postgres=# CREATE DATABASE "mautrix-facebook" OWNER "mautrix-facebook";
postgres=# \q
$ pg_ctl reload -D /var/db/postgres/data13
$ exit
root@synapse:~ #
```

## Install mautrix-facebook
```
pkg install py37-virtualenv olm rust py37-pillow nano

pw adduser mautrix-facebook -d /nonexistent -s /usr/sbin/nologin -c "User for mautrix-facebook bridge"

mkdir /var/db/mautrix-facebook && chown -R mautrix-facebook:mautrix-facebook /var/db/mautrix-facebook

cd /var/db/mautrix-facebook

virtualenv -p /usr/local/bin/python3.7 .

source /var/db/mautrix-facebook/bin/activate.csh

pip install --global-option=build_ext --global-option="-I/usr/local/include" --upgrade python-olm

pip install --upgrade 'mautrix-facebook[all]'

cp example-config.yaml config.yaml
```

Configure mautrix-facebook: `nano config.yaml`:
```
homeserver:
    address: https://exmaple.tld
    domain: example.tld
appservice:
    database: postgres://mautrix-facebook:password@localhost/mautrix-facebook
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
# mkdir /usr/local/etc/mautrix-facebook
# mv config.yaml /usr/local/etc/mautrix-facebook/config.yaml
# chown -R mautrix-facebook:mautrix-facebook /var/db/mautrix-facebook
# chown -R mautrix-facebook:mautrix-facebook /usr/local/etc/mautrix-facebook
# sudo -u mautrix-facebook /var/db/mautrix-facebook/bin/python -m mautrix_facebook -g -c /usr/local/etc/mautrix-facebook/config.yaml -r /usr/local/etc/mautrix-facebook/registration.yaml
```
Add mautrix-facebook to synapse config `nano /usr/local/etc/matrix-synapse/homeserver.yaml`:
```
app_service_config_files:
  - /usr/local/etc/mautrix-facebook/registration.yaml
```
restart synapse:
```
service synapse restart
```
## Create startup script
`touch /usr/local/etc/rc.d/mautrix-facebook && chmod +x /usr/local/etc/rc.d/mautrix-facebook && nano /usr/local/etc/rc.d/mautrix-facebook`:
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
Enable service: `sysrc mautrix_facebook_enable="YES"`

## Dry run verbose mode
```
sudo -u mautrix-facebook /var/db/mautrix-facebook/bin/python -m mautrix_facebook -c /usr/local/etc/mautrix-facebook/config.yaml -r /usr/local/etc/mautrix-facebook/registration.yaml
```
(Ctrl+C to stop)
## Start Service
```
# service mautrix-facebook start
# ps aux
mautrix-facebook 10193 50.4  0.1 110020  66508  -  SJ   00:59   0:00.93 /var/db/mautrix-facebook/bin/python -m mautrix_facebook -c /usr/local/etc/mautrix-facebook/config.yaml -r /usr/local/etc/mautrix-facebook/registration.yaml (python3.7)
root             10192  0.0  0.0  10844   2292  -  SsJ  00:59   0:00.00 daemon: /var/db/mautrix-facebook/bin/python[10193] (daemon)
```
