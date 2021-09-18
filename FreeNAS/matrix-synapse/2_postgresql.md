
[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ **Postgresql** ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Postgresql

### Create a seperate jail for Postgresql
pkg dependancies between [py38-matrix-synapse](https://www.freshports.org/net-im/py-matrix-synapse/) and postgresql can conflict between versions, so to keep things clean lets put our database in its own jail. Minor version upgrades are fine (Such as Postgresql 13.1 to 13.2), but major upgrades require a migration procedure (such as postgresql 13.2 to 14.0). `pkg upgrade -y` is a dangerous move when it comes to databases, be sure you are properly briefed and backed-up before making the attempt!

### Create database dataset & mount to jail

This will allow you to snapshot & backup the database, and keep the data safe if you nuke the jail.
```
/mnt/volume1/apps/synapse/db -> /mnt/volume1/iocage/jails/synapse/root/var/db/postgres/data13
```
Then start the jail

### Switch pkg repo to latest & install
```
# pkg install nano
# mkdir -p /usr/local/etc/pkg/repos/
# nano /usr/local/etc/pkg/repos/FreeBSD.conf
```
```
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```
```
# pkg install postgresql13-server sudo
# sysrc postgresql_enable="YES"
```
### Set permissions and start
```
# chown postgres:postgres /var/db/postgres/data13
# service postgresql start
```

### Initialize database
```
# /usr/local/etc/rc.d/postgresql initdb
# service postgresql start
# sudo -i -u postgres
$ psql
postgres=# CREATE USER "synapse" WITH PASSWORD 'password';
postgres=# CREATE DATABASE synapse ENCODING 'UTF8' LC_COLLATE='C' LC_CTYPE='C' template=template0 OWNER "synapse";
postgres=# \q
$ exit
#
```

### Set database access permission
```
# nano /var/db/postgres/data13/pg_hba.conf
```
```
host    synapse         synapse         192.168.84.79/32        md5
```
### Allow remote connections
```
# nano /var/db/postgres/data13/postgresql.conf
```
```
...
listen_addresses = '*'
port = 5432
...
```
### Reload configuration changes
```
# su -m postgres -c 'pg_ctl reload -D /var/db/postgres/data13'
```
