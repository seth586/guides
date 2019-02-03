[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [**Tor**] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

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

Add user `bitcoin` to the` _tor` group so that bitcoin can read the cookie authentication file in `/var/db/tor`:
```
root@bitcoin:~ # pw usermod bitcoin -G _tor
```
Stop bitcoin service:
```
root@bitcoin:~ # su bitcoin
bitcoin@bitcoin:~ # bitcoin-cli stop
bitcoin@bitcoin:~ # exit
root@bitcoin:~ #
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
root@bitcoin:~ # su bitcoin
bitcoin@bitcoin:~ # bitcoin-cli getnetworkinfo
bitcoin@bitcoin:~ # exit
root@bitcoin:~ #
```
You should see a .onion address listed, verify it works by copy/pasting the .onion address into [bitnodes](https://bitnodes.earn.com) (It may take a while)
