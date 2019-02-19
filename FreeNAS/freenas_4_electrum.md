[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [**Electrum**] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

Electrum Personal Server is a lightweight electrum server to serve an electrum client wallet. The client wallets are compatible with hardware wallets like the ledger nano and TREZOR. So ditch your hardware wallet software, and let use our own node to verify receipt and broadcast our layer 1 transactions, cold storage style!

If you aren't already there, SSH into your freenas box, and switch to your bitcoin console as root:
```
# iocage console bitcoin
#
```

### Prerequisites
Make sure that bitcoind is fully synced before running electrum-personal-server:
```
# bitcoin-cli -datadir=/var/db/bitcoin getblockchaininfo
```

### Install Electrum-Personal-Server

Lets make sure that we have python3 and pip installed:
```
# pkg install py36-pip python3
```
Electrum-personal-server is on github, check for the latest release at https://github.com/chris-belcher/electrum-personal-server/releases .
```
# cd ~
# mkdir /electrum
# cd /electrum
# wget https://github.com/chris-belcher/electrum-personal-server/archive/eps-v0.1.6.tar.gz
# tar xzvf eps-v0.1.6.tar.gz
# rm eps-v0.1.6.tar.gz
```
### Configuration file:
```
# cp electrum-personal-server-eps-v0.1.6/config.cfg_sample /usr/local/etc/electrum.conf
# nano /usr/local/etc/electrum.conf
```
Now we need to add our hardware wallet’s master public keys xpub/ypub/zpub).

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
Under `[bitcoin-rpc]`, change `datadir = /var/db/bitcoin/`

Under `[electrum-serever]`, change `host= 0.0.0.0` to allow remote connections. Change ip_whitelist to your home network subnet, for example, my router assigns IP adresses as `192.168.84.XXX`, so I typed in `192.168.84.0/24` to limit connections to this range. Save and exit nano.

## Install: 
(Note: ignore the suggestion to upgrade pip)
```
# cd electrum-personal-server-eps-v0.1.6
# pip-3.6 install .
# cd ..
# rm -r electrum-personal-server-eps-v0.1.6
```
## First start
```
# /usr/local/bin/electrum-personal-server /usr/local/etc/electrum.conf
```
It will import addresses from each master public key. When complete, electrum-personal-server will exit. Next, if you have transaction history, look up the block height of your oldest transaction, or just start from 1. Then, lets scan the blockchain for those historical transactions:
```
# /usr/local/bin/electrum-personal-server-rescan /usr/local/etc/electrum.conf
```
Lets run it!
```
# /usr/local/bin/electrum-personal-server /usr/local/etc/electrum.conf
```
Now boot up your electrum client, goto Tools>Network>Server, point it at your jail’s ip:50002, it should work!

By default, electrum client will also connect to public electrum servers to get block header info. This reduces your privacy. To prevent this from happening, add `--oneserver --server 192.168.84.123:s` (where the ip address reflects your bitcoin jail) to your command path. In windows, right click on the electrum shortcut, select `properties`, then append `--oneserver --server 192.168.84.123:s` after the executable path `"C:\Program Files (x86)\Electrum\electrum-3.3.2.exe"`. Select `OK` to save and exit.

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
chmod +x /usr/local/etc/rc.d/electrumpersonalserver
```
Lets enable our startup script:
```
# nano /etc/rc.conf
```
Add the following line, then save and exit.
```
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
### How to update electrum personal server:
```
# service electrumpersonalserver stop
# cd /electrum
# wget https://github.com/chris-belcher/electrum-personal-server/archive/eps-v0.1.6.tar.gz
# tar xzvf eps-v0.1.6.tar.gz
# rm eps-v0.1.6.tar.gz
# cd electrum-personal-server-eps-v0.1.6
# pip-3.6 install --upgrade .
# cd ..
# rm -r electrum-personal-server-eps-v0.1.6
# service electrumpersonalserver start
```
Next: [ [lnd](freenas_5_lnd.md) ]
