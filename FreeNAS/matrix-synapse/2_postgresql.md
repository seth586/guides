
[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ **Postgresql** ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Postgresql

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
