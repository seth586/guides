[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [Postgresql](2_postgresql.md) ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - **[ token registration ]** - [ [tor ](6_tor.md)]

## Guide to matrix-synapse server on TrueNAS ![BSDBTC60.png](images/matrix60.png)

### Token based registration
This will allow you to generate invite links, allowing 1 unique registration onto your server per link generated.

## FreeBSD rc.d
In this example pip installed the binary at `/usr/local/bin/matrix-registration`: 

Create user: `pw adduser matrixreg -d /nonexistent -s /usr/sbin/nologin -c "matrix-registration user"`

Create working directory & set permissions: `mkdir /var/db/matrixreg && chown -R matrixreg:matrixreg /var/db/matrixreg`

Create config: `cd /usr/local/etc && fetch https://raw.githubusercontent.com/ZerataX/matrix-registration/master/config.sample.yaml && mv config.sample.yaml matrix-registration.yaml && nano matrix-registration.yaml`:
```
...
server_location: 'http://192.168.1.78:8008'
server_name: 'example.tld'
shared_secret: 'Registration_Shared_Secret'
admin_secret: 'APIAdminPassword'
...
db: 'sqlite:////var/db/matrixreg/db.sqlite3'
...
logging:
 handlers:
   file:
     filename: /var/db/matrixreg/m_reg.log
```

Create rc.d startup script `nano /usr/local/etc/rc.d/matrixreg`:
```
#!/bin/sh
#
# PROVIDE: matrixreg
# REQUIRE:
# KEYWORD:

. /etc/rc.subr

name="matrixreg"
rcvar="${name}_enable"
matrixreg_command="/usr/local/bin/matrix-registration --config-path=/usr/local/etc/matrix-registration.yaml serve"
matrixreg_interpreter="/usr/local/bin/python3.7"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u matrixreg -r -f ${matrixreg_command}"

load_rc_config $name

run_rc_command "$1"
```
Make startup script executable: `chmod +x /usr/local/etc/rc.d/matrixreg`

Enable script on startup: `sysrc matrixreg_enable="YES"`

Start the service: `service matrixreg start`
