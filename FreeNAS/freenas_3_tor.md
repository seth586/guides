[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [**Tor**] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Why Tor?
Goto https://bitnodes.earn.com/dashboard/, or http://nodes.bitcoin-russia.ru and sort by onion protocol. How many nodes do you see? 200-500 as of writing. This is a tragedy. Every bitcoin node should be serving tor connections, there is no reason not to! Without tor, we are publicly proclaiming to our ISP and VPN providers "This home has bitcoins'!

Because of this, nodes that exclusively run on tor have a hard time keeping sync with the network. Lets change that.

### Install Tor

Tor is a communications protocol that anonymizes communications by bouncing encrypted data between relays. It is also slower than TCP/IP. By serving TOR connections, you help other nodes that require the privacy. If you want to run your node over tor only, you can add `/home/bitcoin/.bitcoin/bitcoin.conf` with the line `onlynet=onion`.

Lets install!
```
root@bitcoin:~ # pkg install tor
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
CookieAuthFileGroupReadable 1
CacheDirectoryGroupReadable 1
```
Save (CTRL+O, ENTER), then exit (CTRL+X)

Enable autostart by adding `tor_enable="YES"` to `/etc/rc.conf`.
```
root@bitcoin:~ # nano /etc/rc.conf
tor_enable="YES"
```
Save (Ctrl+O,ENTER) and exit (CTRL+X)

Add user `bitcoin` to the` _tor` group so that bitcoin can read the cookie authentication file in `/var/db/tor`:
```
root@bitcoin:~ # pw usermod bitcoin -G _tor
```
Stop bitcoin service:
```
root@bitcoin:~ # service bitcoind stop
```
Start tor:
```
root@bitcoin:~ # service tor start
```
Wait a few seconds, then start bitcoind:
```
root@bitcoin:~ # service bitcoind start
```
Bitcoin should automatically detect tor and connect. Verify by running bitcoin-cli:
```
root@bitcoin:~ # bitcoin-cli -datadir=/home/bitcoin/.bitcoin getnetworkinfo
```
You should see a .onion address listed, verify it works by copy/pasting the .onion address into [bitnodes](https://bitnodes.earn.com) (It may take a while)

Next: [ [Electrum](freenas_4_electrum.md) ]
