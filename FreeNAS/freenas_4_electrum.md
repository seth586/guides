[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [**Electrum**] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Install Electrum-Personal-Server
Electrum Personal Server is a lightweight electrum server to serve an electrum client wallet. The client wallets are compatible with hardware wallets like the ledger nano. So ditch your ledger live wallet software, and let use our own node to verify and broadcast our transactions!

## Download & Configuration
SSH into your freenas box, and switch to your bitcoin_node console as root:
```
# iocage console bitcoin_node
root@bitcoin:~ #
```
Lets make sure that we have python3 and pip installed:
```
# pkg install py36-pip python3
```
Switch to a bitcoin user session and make an electrum folder.
```
root@bitcoin:~ # su bitcoin
bitcoin@bitcoin:~ % mkdir /home/bitcoin/electrum-personal-server
bitcoin@bitcoin:~ % cd /home/bitcoin/electrum-personal-server
bitcoin@bitcoin:~/electrum-personal-server %
```

Electrum-personal-server is on github, check for the latest release at https://github.com/chris-belcher/electrum-personal-server/releases .
```
bitcoin@bitcoin:~/electrum-personal-server % wget https://github.com/chris-belcher/electrum-personal-server/archive/eps-v0.1.6.tar.gz
bitcoin@bitcoin:~/electrum-personal-server % tar xzvf eps-v0.1.6.tar.gz
bitcoin@bitcoin:~/electrum-personal-server % rm eps-v0.1.6.tar.gz
```
### Configuration file:
```
bitcoin@bitcoin:~/electrum-personal-server % cp electrum-personal-server-eps-v0.1.6/config.cfg_sample config.cfg
bitcoin@bitcoin:~/electrum-personal-server % nano config.cfg
```
Now we need to add our hardware wallet’s master public keys xpub/ypub/zpub).

xpub is for P2PKH (legacy) addresses, which start with a 1

ypub is for P2SH-P2WPKH (segwit), which start with a 3

zpub is for P2WPKH, (bech32 native segwit), which start with bc1

You can find it under Electrum’s menu Wallet>Information.

You 24 word seed can generate all 3, so it might be best to put all 3 in your config.cfg file under `[master-public-keys]`
```
[master-public-keys]
wallet_legacy = xpub.....
wallet_segwit = ypub....
wallet_bech32 = zpub...
```
Under `[bitcoin-rpc]`, change `datadir = /home/bitcoin/.bitcoin/`

Under `[electrum-serever]`, change `host= 0.0.0.0` to allow remote connections. Change ip_whitelist to your home network subnet, for example, my router assigns IP adresses as `192.168.84.XXX`, so I typed in `192.168.84.0/24` to limit connections to this range. Save and exit nano.

Install:
```
# pip-3.6 install --user .
```
First start
```
# /usr/local/bin/electrum-personal-server /usr/local/electrum-personal-server-eps-v0.1.6/config.cfg
```
It will import addresses from each master public key. When complete, electrum-personal-server will exit. Next, if you have transaction history, look up the block height of your oldest transaction. Then, lets scan the blockchain for those historical transactions:
```
# /usr/local/bin/electrum-personal-server-rescan /usr/local/electrum-personal-server-eps-v0.1.6/config.cfg
```
Lets run it!
```
# /usr/local/bin/electrum-personal-server /usr/local/electrum-personal-server-eps-v0.1.6/config.cfg
```
Startup Script
Terminating your SSH will also terminate electrum-personal-server, so lets close it with Ctrl+C, then daemon-zie the process with a rc.d startup script:
```
# nano /etc/rc.d/electrumpersonalserver
```
Copy the following startup script to nano, (follow link to pastebin) then save and exit.

https://pastebin.com/HSFXAbLd
Lets enable our startup script:
```
# nano /etc/rc.conf
```
Add the following line, then save and exit.
```
electrumpersonalserver_enable=YES
```
You should now be able to start and stop electrumpersonalserver as a service.
```
# service electrumpersonalserver start
```
You can run ps auxww in the command line to verify that it is running.
```
# service electrumpersonalserver stop
```
Again, verify that it sucessfully stops with ps auxww . Go ahead and reboot your jail, and check that it is running!

Logging
The startup script uses daemon(8) to run the process in the background and forward all console messages to a log file. We don’t want that log file to grow too large, thankfully FreeBSD has a utility called newsyslog(8), configured by newsyslog.conf
```
# nano /etc/newsyslog.conf
```
Add the following line, then save and exit:
```
/var/log/electrumpersonalserver.log     600  1     100  *     JN
```
Electrum Client
Now boot up your electrum client, goto Tools>Network>Server, point it at your jail’s ip:50002, it should work!
