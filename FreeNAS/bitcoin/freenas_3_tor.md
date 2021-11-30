[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [**Tor & i2p**] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ]

## TrueNASnode - full bitcoin stack deployment guide ![BSDBTC60.png](images/BSDBTC60.png)

### Why use privacy networks?

Tor is a communications protocol that encrypts and anonymizes communications by bouncing encrypted data between relays. It's like using several different VPNs between client and server. TOR also allows you to securely and privately remote connect to your home server with a static address and zero router configuration! Hidden Service Version 3 is not discoverable, so you don't have to worry about exposing ports to the public, as long as you don't share your hidden service onion address!

i2p is similar to tor, however runs as a distributed systyem instead of a curated one. i2p was recently [supported](https://i2pd.readthedocs.io/en/latest/user-guide/FAQ/#how-is-i2p-different-from-tor) by bitcoin core 0.22. We will use the c++ version, [i2pd](https://www.freshports.org/security/i2pd/)

## Install & Configure Tor
```
root@bitcoin:~ # pkg install tor nano
root@bitcoin:~ # nano /usr/local/etc/tor/torrc
```
Uncomment (remove the #) from the following lines:
```
DataDirectory /var/db/tor
ControlPort 9051
CookieAuthentication 1
```
Add the following lines:
```
CookieAuthFile /var/db/tor/control_auth_cookie
CookieAuthFileGroupReadable 1
CacheDirectoryGroupReadable 1
```

Add the following lines to privately serve your remote clients for mobile lightning wallets (`8080`) and (`10009`), and electrum (`50001`):

```
HiddenServiceDir /var/db/tor/remote_connections
HiddenServiceVersion 3
HiddenServicePort 50001 127.0.0.1:50001
HiddenServicePort 8080 127.0.0.1:8080
HiddenServicePort 10009 127.0.0.1:10009
```
Save (CTRL+O, ENTER), then exit (CTRL+X)

Enable autostart by adding `tor_enable="YES"` to `/etc/rc.conf`.
```
# sysrc tor_enable="YES"
```
Save (Ctrl+O,ENTER) and exit (CTRL+X)

Add user `bitcoin` to the` _tor` group so that bitcoin can read the cookie authentication file in `/var/db/tor`, then stop and start the tor and bitcoin service:
```
# pw usermod bitcoin -G _tor
# service bitcoind stop
# service tor start
# service bitcoind start
# bitcoin-cli -datadir=/var/db/bitcoin getnetworkinfo
```
You should see a .onion address listed!

View the private onion address of your new hidden service:
```
# cat /var/db/tor/remote_connections/hostname
myprivateonionaddressocyn4rixm632jid.onion
```

Bootstrap your tor peer discovery by manually adding known tor peers (public list available [here](https://github.com/bitcoin/bitcoin/blob/master/contrib/seeds/nodes_main.txt)) & other informational commands:
```
# bitcoin-cli addnode 2g5qfdkn2vvcbqhzcyvyiitg4ceukybxklraxjnu7atlhd22gdwywaid.onion:8333 add
# bitcoin-cli -datadir=/var/db/bitcoin -addrinfo
# bitcoin-cli -datadir=/var/db/bitcoin -netinfo 4
```

### How to upgrade tor:
```
# service tor stop
# pkg update && pkg upgrade tor
# service tor start
```

## Install & Configure i2pd

Bitcoind's use of i2pd is well docuemnted [here](https://github.com/bitcoin/bitcoin/blob/master/doc/i2p.md)
```
# pkg install i2pd
# sysrc i2pd_enable="YES"
# nano /usr/local/etc/i2pd/i2pd.conf
```
The following configuration changes enables SAM protocol hidden service for bitcoind & turns on the web-ui for monitoring. This is not the complete configuration file, these are just the fields that you should change. the [http] `address =` field should be your jail IP. This is the web-ui to monitor your i2pd session.
```
...
loglevel = none
[http]
enabled = true
address = 192.168.84.21
port = 7070
[httpproxy]
enabled = false
[socksproxy]
enabled = false
[sam]
enabled = true
[bob]
enabled = false
[i2cp]
enabled = false
[i2pcontrol]
enabled = false
[upnp]
enabled = false
```
Save (CTRL+O, ENTER) and exit (CTRL+X). 

Edit bitcoin.conf `nano /usr/local/etc/bitcoin.conf` and add the following lines:

```
i2psam=127.0.0.1:7656
i2pacceptincoming=1
```
Save (CTRL+O, ENTER) and exit (CTRL+X). 

### Optional: Relay

If you want to donate bandwidth as a i2p relay, add the following, pick a random `port =` between 10000 and 65000 to share 256KB/s:

`nano /usr/local/etc/i2pd/i2pd.conf`:
```
port = 12345
bandwidth = o
```
Port forward TCP+UDP `port =` from your router to your bitcoin jail `port =`. 

### Start & verify
```
# service bitcoind stop
# service i2pd start
# service bitcoind start
# bitcoin-cli getnetworkinfo
```
Open a browser and navigate to your bitcoin jail and port 7070, ex: `http://192.168.84.21:7070/`. Network status should read: Firewalled if you are not running a relay, and OK if you are running a relay. Click `SAM Sessions`. You shoud see bitcoind's SAM session! You can go back and disable the [http] section in `/usr/local/etc/i2pd/i2pd.conf` if you dont want to monitor, or add password credentials to secure the monitoring & configuration interface.

### Bootstrap i2p peers
Find a list of publically maintained addresses [here](https://github.com/bitcoin/bitcoin/blob/master/contrib/seeds/nodes_main.txt)
```
# bitcoin-cli addnode 4hllr6w55mbtemb3ebvlzl4zj6qke4si7zcob5qdyg63mjgq624a.b32.i2p:0 add
# bitcoin-cli -datadir=/var/db/bitcoin -addrinfo
# bitcoin-cli -datadir=/var/db/bitcoin -netinfo 4
```

### How to upgrade i2p:
```
# service i2pd stop
# pkg update && pkg upgrade i2pd -y
# service i2pd start
```

Next: [ [Electrum](freenas_4_electrum.md) ]
