
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
# pkg install -y py39-matrix-synapse
# sysrc synapse_enable="YES"
```

### Configuration

Familiarize yourself with the [documentation](https://matrix-org.github.io/synapse/latest/setup/installation.html) as everyone must configure to their individual needs.

Create `homeserver.yaml`, Replace `domain.tld` with the domain of your matrix-synapse server:
```
# /usr/local/bin/python3.9 -B -m synapse.app.homeserver -c /usr/local/etc/matrix-synapse/homeserver.yaml --generate-config -H domain.tld --report-stats no
# nano /usr/local/etc/matrix-synapse/homeserver.yaml
```

### Upgrade Synapse

Make SURE to read upgrade instructions [here](https://github.com/matrix-org/synapse/blob/develop/docs/upgrade.md)
```

```
