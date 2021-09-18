
[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [postgresql](2_postgresql.md) ] - [ **synapse** ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Synapse

### Switch jails and Install
```
# exit
root@truenas[~]# iocage console synapse
# pkg install -y nano
# mkdir -p /usr/local/etc/pkg/repos/
# nano /usr/local/etc/pkg/repos/FreeBSD.conf
```
```
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```
Save (CTRL+o, ENTER) and exit (CTRL+x)

```
# pkg install -y py38-matrix-synapse
# sysrc synapse_enable="YES"
```

### Configure
There is a lot to unpack here, these are my minimum lines to edit. Search for the following lines (CTRL+W) and make appropriate entries
```
nano /usr/local/etc/matrix-synapse/homeserver.yaml
```
