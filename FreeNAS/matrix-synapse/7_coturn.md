[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

**[Intro]** - [ [Jail Creation](1_jail.md) ] - [ [Postgresql](2_postgresql.md) ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - **[coturn]** - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Guide to matrix-synapse server on TrueNAS ![BSDBTC60.png](images/matrix60.png)

### Coturn

#### Poor mans ddns
coturn is designed to be utilized without a NAT and with a static IP address. You and I are likely using a NAT (router) and a dynamic ip address. This script will be called every 5 minutes as a cronjob, ask the router's upnp daemon for the WAN ip address, and if its different than whats it reads in `turnserver.conf`, will edit `turnserver.conf` with the new IP and restart coturn.

Install `miniupnpd` on your router, and enable the service. Or, if you're not using an open source operating system on your router, poray to the gods the manufacturer set up upnp correctly...

`root@turnserver:~ #` `pkg install upnpc`

`root@turnserver:~ #` `touch /usr/local/etc/coturn_ext_ip.sh && chmod +x /usr/local/etc/coturn_ext_ip.sh && nano /usr/local/etc/coturn_ext_ip.sh`:
```
#!/bin/csh
set current_external_ip_config = `cat /usr/local/etc/turnserver.conf | grep "^external-ip" | cut -d'=' -f2`
set current_external_ip = `upnpc -s | grep "^ExternalIPAddress" | cut -d'=' -f2 | cut -d' ' -f2`

if ( "$current_external_ip_config" != "$current_external_ip" ) then
  sed -i '' "s/^external-ip=.*/external-ip="$current_external_ip"/" /usr/local/etc/turnserver.conf
  service turnserver restart
endif
```

`root@turnserver:~ #` `crontab -e`:
```
*/5  * * * * /usr/local/etc/coturn_ext_ip.sh
```
