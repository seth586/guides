[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/FreeNAS/matrix-synapse/9_bridges.md) ]

## Twitter bridge

### Configure database
Allow connection to database `# nano /var/db/postgres/data13/pg_hba.conf`:
```
host    mautrix-twitter  mautrix-twitter  127.0.0.1/32         password
```

### Create database:
```
root@synapse:~ # sudo -i -u postgres
$ psql
postgres=# CREATE USER "mautrix-twitter" WITH PASSWORD 'password';
postgres=# CREATE DATABASE "mautrix-twitter" OWNER "mautrix-twitter";
postgres=# \q
$ pg_ctl reload -D /var/db/postgres/data13
$ exit
root@synapse:~ #
```

### Install mautrix-twitter
```
# pkg install py37-virtualenv olm rust py37-pillow nano

# pw adduser mautrix-twitter -d /nonexistent -s /usr/sbin/nologin -c "User for mautrix-twitter bridge"

# mkdir /var/db/mautrix-twitter && chown -R mautrix-twitter:mautrix-twitter /var/db/mautrix-twitter

# cd /var/db/mautrix-twitter

# virtualenv -p /usr/local/bin/python3.7 .

# source /var/db/mautrix-twitter/bin/activate.csh

# pip install --global-option=build_ext --global-option="-I/usr/local/include" --upgrade python-olm

# pip install --upgrade 'mautrix-twitter[all]'

# cp example-config.yaml config.yaml
```

### Configure mautrix-twitter: 

`# nano config.yaml`:
```
homeserver:
    address: https://example.tld
    domain: example.tld
appservice:
    database: postgres://mautrix-twitter:password@localhost/mautrix-twitter
bridge:
    permissions:
        example.tld: user
        `@admin:example.tld`: admin
logging:
    handlers:
        file:
            filename: /var/db/mautrix-twitter/mautrix-twitter.log
```

Generate `registration.yaml`:
```
# mkdir /usr/local/etc/mautrix-twitter
# mv config.yaml /usr/local/etc/mautrix-twitter/config.yaml
# chown -R mautrix-twitter:mautrix-twitter /var/db/mautrix-twitter
# chown -R mautrix-twitter:mautrix-twitter /usr/local/etc/mautrix-twitter
# sudo -u mautrix-twitter /var/db/mautrix-twitter/bin/python -m mautrix_twitter -g -c /usr/local/etc/mautrix-twitter/config.yaml -r /usr/local/etc/mautrix-twitter/registration.yaml
```
Add mautrix-twitter to synapse config `nano /usr/local/etc/matrix-synapse/homeserver.yaml`:
```
app_service_config_files:
  - /usr/local/etc/mautrix-twitter/registration.yaml
```
restart synapse:
```
service synapse restart
```
### Create startup script
`touch /usr/local/etc/rc.d/mautrix-twitter && chmod +x /usr/local/etc/rc.d/mautrix-twitter && nano /usr/local/etc/rc.d/mautrix-twitter`:
```
#!/bin/sh
#
# PROVIDE: mautrix_twitter
# REQUIRE:
# KEYWORD:

. /etc/rc.subr
name="mautrix_twitter"
rcvar="mautrix_twitter_enable"
mautrix_twitter_command="/var/db/mautrix-twitter/bin/python -m mautrix_twitter -c /usr/local/etc/mautrix-twitter/config.yaml -r /usr/local/etc/mautrix-twitter/registration.yaml"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u mautrix-twitter -r -f ${mautrix_twitter_command}"

load_rc_config $name
: ${mautrix_twitter_enable:=no}

run_rc_command "$1"
```
Enable service: `sysrc mautrix_twitter_enable="YES"`

### Dry run verbose mode
```
sudo -u mautrix-twitter /var/db/mautrix-twitter/bin/python -m mautrix_twitter -c /usr/local/etc/mautrix-twitter/config.yaml -r /usr/local/etc/mautrix-twitter/registration.yaml
```
(Ctrl+C to stop)
### Start Service
```
# service mautrix-twitter start
# ps aux
mautrix-twitter 10193 50.4  0.1 110020  66508  -  SJ   00:59   0:00.93 /var/db/mautrix-twitter/bin/python -m mautrix_twitter -c /usr/local/etc/mautrix-twitter/config.yaml -r /usr/local/etc/mautrix-twitter/registration.yaml (python3.7)
root             10192  0.0  0.0  10844   2292  -  SsJ  00:59   0:00.00 daemon: /var/db/mautrix-twitter/bin/python[10193] (daemon)
```

[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/FreeNAS/matrix-synapse/9_bridges.md) ]
