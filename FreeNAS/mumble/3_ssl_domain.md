[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [murmur](2_murmur.md) ] - [ **SSL & Domain** ] - [ [Basic ACL Config](4_acl.md) ]

## Guide to Mumble server (murmur) on FreeNAS/TrueNAS
### SSL & Domain Configuration
In the [Guide to a self hosted wordpress website on FreeNAS/TrueNAS](https://github.com/seth586/guides/tree/master/FreeNAS/webserver), I specify in detail how to create a [reverse proxy](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) to serve multiple websites. On this reverse-proxy jail, nginx monitors website requests on the default port 80 for http and 443 for https, and sends those requests to the appropriate jail for our multiple websites we are hosting now or in the future.

Murmur is not using a shared port like websites use on 80 and 443 by default, so we will not need nginx to reverse proxy port 64738, the firewall rule we created during [Jail Creation](1_jail_creation.md) will forward TCP and UDP requests on `(public IP address):64738` to our `(mumble jail IP address):64738`.

However the [reverse-proxy jail guide](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) also handles our domain certificate request and renewals, so for the sake of configuring and maintaining a domain and SSL/TLS certificates for mumble, please complete steps 1-7 on that page, then come back here. 

## Mount Points for SSL/TLS keys
```
# mkdir /usr/local/etc/certs
```

We need to share the public and private SSL/TLS keys in our `reverse-proxy` jail with our `mumble` jail, so log in to FreeNAS' web-ui and "stop" the mumble jail. Click "mount points". Click "Actions ▼", "Add".

Source: `/mnt/volume0/iocage/jails/reverse-proxy/root/usr/local/etc/letsencrypt/live/example.com`
Destination: `/mnt/volume0/iocage/jails/mumble/root/usr/local/etc/certs`
