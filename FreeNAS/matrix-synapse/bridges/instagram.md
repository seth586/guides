[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/FreeNAS/matrix-synapse/9_bridges.md) ]

## Instagram bridge

### Configure database
Allow connection to database `# nano /var/db/postgres/data13/pg_hba.conf`:
```
host    mautrix-instagram  mautrix-instagram  127.0.0.1/32         password
```

### Create database:
```
root@synapse:~ # sudo -i -u postgres
$ psql
postgres=# CREATE USER "mautrix-instagram" WITH PASSWORD 'password';
postgres=# CREATE DATABASE "mautrix-instagram" OWNER "mautrix-instagram";
postgres=# \q
$ pg_ctl reload -D /var/db/postgres/data13
$ exit
root@synapse:~ #
```

### Install mautrix-instagram
```
# pkg install py37-virtualenv olm rust py37-pillow nano

# pw adduser mautrix-instagram -d /nonexistent -s /usr/sbin/nologin -c "User for mautrix-instagram bridge"

# mkdir /var/db/mautrix-instagram && chown -R mautrix-instagram:mautrix-instagram /var/db/mautrix-instagram

# cd /var/db/mautrix-instagram

# virtualenv -p /usr/local/bin/python3.7 .

# source /var/db/mautrix-instagram/bin/activate.csh

# pip install --global-option=build_ext --global-option="-I/usr/local/include" --upgrade python-olm

# pip install --upgrade 'https://github.com/tulir/mautrix-instagram/tarball/master#egg=mautrix-instagram[all]'

# cp example-config.yaml config.yaml
```

### Configure mautrix-instagram: 

At minimum change these values `# nano config.yaml`:
```
homeserver:
    address: https://example.tld
    domain: example.tld
appservice:
    database: postgres://mautrix-instagram:password@localhost/mautrix-instagram
bridge:
    permissions:
        example.tld: user
        `@admin:example.tld`: admin
logging:
    handlers:
        file:
            filename: /var/db/mautrix-instagram/mautrix-instagram.log
```

Generate `registration.yaml`:
```
# mkdir /usr/local/etc/mautrix-instagram
# mv config.yaml /usr/local/etc/mautrix-instagram/config.yaml
# chown -R mautrix-instagram:mautrix-instagram /var/db/mautrix-instagram
# chown -R mautrix-instagram:mautrix-instagram /usr/local/etc/mautrix-instagram
# sudo -u mautrix-instagram /var/db/mautrix-instagram/bin/python -m mautrix_instagram -g -c /usr/local/etc/mautrix-instagram/config.yaml -r /usr/local/etc/mautrix-instagram/registration.yaml
```
Add mautrix-instagram to synapse config `nano /usr/local/etc/matrix-synapse/homeserver.yaml`:
```
app_service_config_files:
  - /usr/local/etc/mautrix-instagram/registration.yaml
```
restart synapse:
```
service synapse restart
```
### Create startup script
`touch /usr/local/etc/rc.d/mautrix-instagram && chmod +x /usr/local/etc/rc.d/mautrix-instagram && nano /usr/local/etc/rc.d/mautrix-instagram`:
```
#!/bin/sh
#
# PROVIDE: mautrix_instagram
# REQUIRE:
# KEYWORD:

. /etc/rc.subr
name="mautrix_instagram"
rcvar="mautrix_instagram_enable"
mautrix_instagram_command="/var/db/mautrix-instagram/bin/python -m mautrix_instagram -c /usr/local/etc/mautrix-instagram/config.yaml -r /usr/local/etc/mautrix-instagram/registration.yaml"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u mautrix-instagram -r -f ${mautrix_instagram_command}"

load_rc_config $name
: ${mautrix_instagram_enable:=no}

run_rc_command "$1"
```
Enable service: `sysrc mautrix_instagram_enable="YES"`

### Dry run verbose mode
```
sudo -u mautrix-instagram /var/db/mautrix-instagram/bin/python -m mautrix_instagram -c /usr/local/etc/mautrix-instagram/config.yaml -r /usr/local/etc/mautrix-instagram/registration.yaml
```
(Ctrl+C to stop)
### Start Service
```
# service mautrix-instagram start
# ps aux
mautrix-instagram 10193 50.4  0.1 110020  66508  -  SJ   00:59   0:00.93 /var/db/mautrix-instagram/bin/python -m mautrix_instagram -c /usr/local/etc/mautrix-instagram/config.yaml -r /usr/local/etc/mautrix-instagram/registration.yaml (python3.7)
root             10192  0.0  0.0  10844   2292  -  SsJ  00:59   0:00.00 daemon: /var/db/mautrix-instagram/bin/python[10193] (daemon)
```

[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/FreeNAS/matrix-synapse/9_bridges.md) ]
