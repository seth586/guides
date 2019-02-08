[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [**Electrum**] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈


#### 🚧🚧🚧THIS SECTION IS STILL UNDER CONSTRUCTION, DO NOT USE!🚧🚧🚧

If you aren't already there, SSH into your freenas box, and switch to your bitcoin console as root:
```
# iocage console bitcoin
#
```

### Prerequisites
Make sure that bitcoind is fully synced before running electrum-personal-server:
```
# bitcoin-cli -datadir=/home/bitcoin/.bitcoin getblockchaininfo
```

### Install Electrum-Personal-Server
Electrum Personal Server is a lightweight electrum server to serve an electrum client wallet. The client wallets are compatible with hardware wallets like the ledger nano. So ditch your ledger live wallet software, and let use our own node to verify receipt and broadcast our layer 1 transactions!

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
cp electrum-personal-server-eps-v0.1.6/config.cfg_sample config.cfg
nano config.cfg
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
Under `[bitcoin-rpc]`, change `datadir = /home/bitcoin/.bitcoin/`

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
# /usr/local/bin/electrum-personal-server /electrum/config.cfg
```
It will import addresses from each master public key. When complete, electrum-personal-server will exit. Next, if you have transaction history, look up the block height of your oldest transaction, or just start from 1. Then, lets scan the blockchain for those historical transactions:
```
# /usr/local/bin/electrum-personal-server-rescan /electrum/config.cfg
```
Lets run it!
```
# /usr/local/bin/electrum-personal-server /electrum/config.cfg
```
Now boot up your electrum client, goto Tools>Network>Server, point it at your jail’s ip:50002, it should work!

## Startup Script
Terminating your SSH will also terminate electrum-personal-server, so lets close it with Ctrl+C, then daemon-zie the process with a rc.d startup script:
```
# nano /usr/local/etc/rc.d/electrumpersonalserver
```
Copy the following startup script to nano, then save and exit.
```
#!/bin/sh
# REQUIRE: LOGIN cleanvar bitcoind
 
. /etc/rc.subr
 
name=electrumpersonalserver
rcvar=electrumpersonalserver_enable
 
command="/usr/local/bin/electrum-personal-server"
command_args="/usr/local/electrum-personal-server-eps-v0.1.6/config.cfg"
logfile="/var/log/${name}.log"
pidfile="/var/run/${name}.pid"
sig_stop=-KILL
 
start_cmd="${name}_start"
stop_cmd="${name}_stop"
status_cmd="${name}_status"
 
electrumpersonalserver_start()
{
        /usr/sbin/daemon -f -p ${pidfile} ${command} $command_args
        if [ $? -ne 0 ]; then
                echo "Error starting ${name}."
                exit 1
        fi
    echo "Starting ${name}."
}
 
electrumpersonalserver_stop()
{
    echo "stopping ${name}"
    kill `cat ${pidfile}`
}
 
electrumpersonalserver_status()
{
    if [ -e ${pidfile} ] ; then
        if ps `cat ${pidfile}` > /dev/null ; then
            echo "${name} is running"
            return
        fi
    fi
    echo "${name} is not running"
}
load_rc_config $name
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
electrumpersonalserver_enable=YES
```
You should now be able to start and stop electrumpersonalserver as a service.
```
# service electrumpersonalserver start
```
You can run `ps aux` in the command line to verify that it is running.
```
# service electrumpersonalserver stop
```
Again, verify that it sucessfully stops with `ps aux` . Go ahead and reboot your jail, and check that it is running!

Next: [ [lnd](freenas_5_lnd.md) ]
