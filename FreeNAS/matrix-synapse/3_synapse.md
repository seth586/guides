
[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [postgresql](2_postgresql.md) ] - [ **synapse** ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Synapse

### Switch jails and Install
```
exit
iocage console synapse
pkg install nano -y
mkdir -p /usr/local/etc/pkg/repos/
nano /usr/local/etc/pkg/repos/FreeBSD.conf
```
```
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```

