
[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ **Postgresql** ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Postgresql

### Create a seperate jail for Postgresql
pkg dependancies between matrix-synapse38 and postgresql can conflict between versions, so to keep things clean lets put our database in its own jail. Minor version upgrades are fine (Such as Postgresql 13.1 to 13.2), but major upgrades require a migration procedure (such as postgresql 13.2 to 14.0).

As a bonus, this database installation can serve other jails as well (nextcloud, mempool.space, etc)

### Switch pkg repo to latest
```
pkg install nano
mkdir -p /usr/local/etc/pkg/repos/
nano /usr/local/etc/pkg/repos/FreeBSD.conf
```
```
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```
```
pkg install postgresql13-server
sysrc postgresql_enable="YES"
```
