[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [Postgresql](2_postgresql.md) ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - **[ token registration ]** - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ]- [ [jitsi](8_jitsi.md) ]

## Guide to matrix-synapse server on TrueNAS ![BSDBTC60.png](images/matrix60.png)

### Token based registration
https://github.com/ZerataX/matrix-registration

This will allow you to generate invite links, allowing 1 unique registration onto your server per link generated.

## Create database for matrixreg (in synapsedb jail)
```
iocage console synapsedb
# sudo -i -u postgres
$ psql
postgres=# CREATE USER "matrixreg" WITH PASSWORD 'password';
postgres=# CREATE DATABASE matrixreg ENCODING 'UTF8' LC_COLLATE='C' LC_CTYPE='C' template=template0 OWNER "matrixreg";
postgres=# \q
$ exit
#
```
## Set database access permission (in synapsedb jail)
```
nano /var/db/postgres/data13/pg_hba.conf
```
```
host    matrixreg         matrixreg         192.168.84.79/32        md5
```

## Install (in synapse jail)
```
exit
iocage console synapse
pkg install py38-pip
pip install matrix-registration==1.0.0.dev7
```

## Config
In this example pip installed the binary at `/usr/local/bin/matrix-registration`: 

Create user: `pw adduser matrixreg -d /nonexistent -s /usr/sbin/nologin -c "matrix-registration user"`

Create working directory & set permissions: `mkdir /var/db/matrixreg`

Create config: `cd /usr/local/etc && fetch https://raw.githubusercontent.com/ZerataX/matrix-registration/master/config.sample.yaml && mv config.sample.yaml matrix-registration.yaml && nano matrix-registration.yaml`:
```
...
server_location: 'http://192.168.84.79:8008'
server_name: 'example.tld'
shared_secret: 'Registration_Shared_Secret'
admin_api_shared_secret: 'APIAdminPassword'
...
db: 'postgresql://matrixreg:matrixreg@192.168.84.78:5432/matrixreg'
host: '192.168.84.79'
...
logging:
 handlers:
   file:
     filename: /var/db/matrixreg/m_reg.log
```
## Test with a verbose run:
```
/usr/local/bin/python3.8 /usr/local/bin/matrix-registration --config-path=/usr/local/etc/matrix-registration.yaml serve
waitress - INFO - Serving on http://192.168.84.79:5000
```
Ctrl+C to stop

## Create rc.d startup script `nano /usr/local/etc/rc.d/matrixreg`:
```
#!/bin/sh
#
# PROVIDE: matrixreg
# REQUIRE: LOGIN postgresql synapse
# KEYWORD:

. /etc/rc.subr

name="matrixreg"
rcvar="${name}_enable"
matrixreg_command="/usr/local/bin/matrix-registration --config-path=/usr/local/etc/matrix-registration.yaml serve"
matrixreg_interpreter="/usr/local/bin/python3.8"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u matrixreg -r -f ${matrixreg_command}"

load_rc_config $name

run_rc_command "$1"
```
Make startup script executable: `chmod +x /usr/local/etc/rc.d/matrixreg`

Enable script on startup: `sysrc matrixreg_enable="YES"`

Set permissions: `chown -R matrixreg:matrixreg /var/db/matrixreg`

Start the service: `service matrixreg start`
