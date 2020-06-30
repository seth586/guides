[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [murmur](2_murmur.md) ] - [ **SSL & Domain** ] - [ [Basic ACL Config](4_acl.md) ]

## Guide to Mumble server (murmur) on FreeNAS/TrueNAS
### SSL & Domain Configuration
In the [Guide to a self hosted wordpress website on FreeNAS/TrueNAS](https://github.com/seth586/guides/tree/master/FreeNAS/webserver), I specify in detail how to create a [reverse proxy](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) to serve multiple websites. On this reverse-proxy jail, nginx monitors website requests on the default port 80 for http and 443 for https, and sends those requests to the appropriate jail for our multiple websites we are hosting now or in the future.

Murmur is not using a shared port like websites use on 80 and 443 by default, so we will not need nginx to reverse proxy port 64738, the firewall rule we created during [Jail Creation](1_jail_creation.md) will forward TCP and UDP requests on `(public IP address):64738` to our `(mumble jail IP address):64738`.

However the [reverse-proxy jail guide](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) also handles our domain certificate request and renewals, so for the sake of configuring and maintaining a domain and SSL/TLS certificates for mumble, and for future website projects on our server, this mumble guide will require the reverse proxy jail to be created and utilized.

## Configure Dynamic DNS & Certbot
Please complete steps 1-5 (minimum!) on the [reverse-proxy tutorial](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) if you have not already done so, then come back here. 

## Mount Points for SSL/TLS keys
```
# mkdir /usr/local/etc/certs
```

We need to share the public and private SSL/TLS keys in our `reverse-proxy` jail with our `mumble` jail, so log in to FreeNAS' web-ui and "stop" the mumble jail. Click "mount points". Click "Actions ▼", "Add".

Remember to replace `example.com` with your own domain.

Source: `/mnt/volume0/iocage/jails/reverse-proxy/root/usr/local/etc/letsencrypt`

Destination: `/mnt/volume0/iocage/jails/mumble/root/usr/local/etc/certs`

Check the "Read Only" box. Click "Save".

Start the jail and SSH in.

```
# cd /usr/local/etc/certs
# ll
drwxr-xr-x  3 root  wheel   3 Feb 17 05:23 accounts/
drwxr-xr-x  3 root  certs   3 Jun 30 02:14 archive/
drwxr-xr-x  2 root  wheel  12 Jun 20 01:09 csr/
drwx------  2 root  wheel  12 Jun 20 01:09 keys/
drwxr-xr-x  3 root  certs   4 May 11 21:16 live/
drwxr-xr-x  2 root  wheel   5 Jun 20 01:10 renewal/
drwxr-xr-x  5 root  wheel   5 Feb 17 05:23 renewal-hooks/
```
## Add certs group and add murmur to certs group
```
# pw groupadd certs
# pw groupmod certs -M murmur
# pw groupshow certs
certs:*:1001:murmur
# id murmur
uid=338(murmur) gid=338(murmur) groups=338(murmur),1001(certs)
```

## Configure murmur for SSL/TLS certificates
```
nano /usr/local/etc/murmur.ini
```
Edit the following lines:
```
sslCert=/usr/local/etc/certs/live/example.com/fullchain.pem
sslKey=/usr/local/etc/certs/live/example.com/privkey.pem
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

Now start murmur:
```
# service murmur start && tail -f /var/log/murmur/murmur.log
Starting murmur.
```

