[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [**Electrum**] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

This guide is not required for running lightning, but is still a great way to run a trusted electrum server for your hardware wallets!

Electrum Personal Server is a lightweight electrum server to serve an electrum client wallet. The client wallets are compatible with hardware wallets like the ledger nano s, trezor, and coldcard. 

Electrum Personal Server is lightweight and will only monitor addresses and xpubs you specify. If you want to run a heavier version (+40GB as of writing) that can look up any transaction or xpub, check out the [extras](extras.md) page on instructions to install [Electrs](https://github.com/romanz/electrs) instead.

If you aren't already there, SSH into your freenas box, and switch to your bitcoin console as root:
```
# iocage console bitcoin
#
```

### Prerequisites
Make sure that bitcoind is fully synced before running electrum-personal-server, then install some utilities to fetch the source code:
```
# bitcoin-cli -datadir=/var/db/bitcoin getblockchaininfo
# pkg install py36-pip python3 wget ca_root_nss nano
```
Electrum-personal-server is on github, check for the latest release at https://github.com/chris-belcher/electrum-personal-server/releases .
```
# cd ~
# wget https://github.com/chris-belcher/electrum-personal-server/archive/electrum-personal-server-v0.1.7.tar.gz
# tar xvf electrum-personal-server-v0.1.7.tar.gz
# rm electrum-personal-server-v0.1.7.tar.gz
```
### Configuration file:
```
# cp electrum-personal-server-electrum-personal-server-v0.1.7/config.ini_sample /usr/local/etc/electrum.conf
# nano /usr/local/etc/electrum.conf
```
Now we need to add our hardware wallet’s master public keys (xpub/ypub/zpub) under `[master-public-keys]`.

xpub is for P2PKH (legacy) addresses, where generated addresses start with a 1

ypub is for P2SH-P2WPKH (segwit), where generated addresses start with a 3

zpub is for P2WPKH, (bech32 native segwit), where generated addresses start with bc1

You can find it under Electrum’s menu Wallet>Information.

You 24 word seed can generate all 3, so it might be best to put all 3 in your config.cfg file under `[master-public-keys]`
```
[master-public-keys]
wallet_legacy = xpub.....
wallet_segwit = ypub....
wallet_bech32 = zpub...
```
Under `[bitcoin-rpc]`, change `datadir = /var/db/bitcoin`

Under `[electrum-server]`, change `host = ` to `host = 127.0.0.1`

Save (CTRL+O, ENTER) and exit (CTRL+X) nano.

## Install: 
(Note: ignore the suggestion to upgrade pip)
```
# cd electrum-personal-server-electrum-personal-server-v0.1.7
# pip-3.6 install .
# cd ..
# rm -r electrum-personal-server-electrum-personal-server-v0.1.7
```
## First start
```
# /usr/local/bin/electrum-personal-server /usr/local/etc/electrum.conf
```
It will import addresses from each master public key. When complete, electrum-personal-server will exit. Next, if you have transaction history, look up the block height of your oldest transaction, or just start from 1. Then, lets scan the blockchain for those historical transactions:
```
# /usr/local/bin/electrum-personal-server --rescan /usr/local/etc/electrum.conf
```
Lets run it!
```
# /usr/local/bin/electrum-personal-server /usr/local/etc/electrum.conf
```
Now on your client machine, make sure tor browser is open and connected. In windows, right click on the electrum shortcut, select `properties`, then change the shortcut (keep the program path) `"C:\Program Files (x86)\Electrum\electrum-3.3.4.exe" -1 -s myprivateonionaddressocyn4rixm632jid.onion:50002:s`. Select `OK` to save and exit. Now start your electrum client, it should connect, even from a remote internet connection!

## Startup Script
Terminating your SSH will also terminate electrum-personal-server, so lets close it with Ctrl+C, then run the process supervised with `daemon` called at startup with a rc.d service script:
```
# nano /usr/local/etc/rc.d/electrumpersonalserver
```
Copy the following startup script to nano, then save and exit.
```
#!/bin/sh
#
# PROVIDE: electrumpersonalserver
# REQUIRE: bitcoind
# KEYWORD:

. /etc/rc.subr

name="electrumpersonalserver"
rcvar="electrumpersonalserver_enable"
electrumpersonalserver_command="/usr/local/bin/electrum-personal-server /usr/local/etc/electrum.conf"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u bitcoin -r -f ${electrumpersonalserver_command}"

load_rc_config $name
: ${electrumpersonalserver_enable:=no}

run_rc_command "$1"

```
Make the startup script executable:
```
# chmod +x /usr/local/etc/rc.d/electrumpersonalserver
```
Lets enable our startup script. Add the following line, then save and exit.
```
# nano /etc/rc.conf
electrumpersonalserver_enable="YES"
```
You should now be able to start and stop electrumpersonalserver as a service.
```
# service electrumpersonalserver start
```
You can run `ps aux` in the command line to verify that it is running.
```
# service electrumpersonalserver stop
```
Again, verify that it sucessfully stops with `ps aux` . Go ahead and reboot your jail, and check that it is running:
```
# exit
root@freenas[~]# iocage restart bitcoin
root@freenas[~]# iocage console bitcoin
```

### Usage Notes
If you need to add another wallet, edit `nano /usr/local/etc/electrum.conf` to add the new xpubs, then run the following commands:
```
# service electrumpersonalserver stop
# /usr/local/bin/electrum-personal-server /usr/local/etc/electrum.conf
# /usr/local/bin/electrum-personal-server --rescan /usr/local/etc/electrum.conf
# service electrumpersonalserver start

```
### How to update electrum personal server:
```
# service electrumpersonalserver stop
# cd ~
# wget https://github.com/chris-belcher/electrum-personal-server/archive/electrum-personal-server-v0.1.7.tar.gz
# tar xvf electrum-personal-server-v0.1.7.tar.gz
# rm electrum-personal-server-v0.1.7.tar.gz
# cd electrum-personal-server-electrum-personal-server-v0.1.7
# pip-3.6 install --upgrade .
# cd ..
# rm -r electrum-personal-server-electrum-personal-server-v0.1.7
# service electrumpersonalserver start
```
Next: [ [lnd](freenas_5_lnd.md) ]
