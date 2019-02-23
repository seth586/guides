[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [**lnd**] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Install Lightning Lab's LND

If not already there, SSH into your freenas box as root, then switch to your bitcoin jail:
```
root@freenas[~] # iocage console bitcoin
```

Check [LND's github repo](https://github.com/lightningnetwork/lnd/releases) for the latest release, make sure you select the correct binaries for your processor and operating system. (amd64 is for amd and intel processors)
```
# cd ~
# wget https://github.com/lightningnetwork/lnd/releases/download/v0.5.2-beta/lnd-freebsd-amd64-v0.5.2-beta.tar.gz
# tar -xzf lnd-freebsd-amd64-v0.5.2-beta.tar.gz
# cd lnd-freebsd-amd64-v0.5.2-beta
# install -m 0755 -o root -g wheel lnd lncli /usr/local/bin
# cd ~
# rm -r lnd-freebsd-amd64-v0.5.2-beta
# nano /usr/local/etc/lnd.conf
```
### LND Configuration
Read up on configuration options [here](https://github.com/lightningnetwork/lnd/blob/master/sample-lnd.conf). 
Add the following lines to your `lnd.conf` file:
```
[Application Options]
datadir=/var/db/lnd/data
logdir=/var/db/lnd/logs
maxlogfiles=1
maxlogfilesize=10
tlscertpath=/var/db/lnd
tlskeypath=/var/db/lnd

externalip=xxx.xxx.xxx.xxx
listen=0.0.0.0:9735
restlisten=0.0.0.0:8082
alias=thisisthenamethatblockexplorerswillshow


[Bitcoin]
bitcoin.active=1
bitcoin.mainnet=1
bitcoin.node=bitcoind
bitcoin.basefee=1000
bitcoin.feerate=2500

[Bitcoind]
bitcoind.dir=/var/db/bitcoin
```
### Configuration Notes
Fees. You may have to pay fees to other nodes when you rebalance channels, don't operate at a loss!  
`bitcoin.basefee=1000` = Fee of 1.000 satoshi per payment forwarded
`bitcoin.feerate=2500` = Fee of 0.25% of amount forwarded

`externalip=` = Set if you have a static IP address assigned by your ISP. If your ip address changes with this set, inbound channels will lose connection with you. If your ISP assigns a dynamic ip address, use `nat=true` instead of `externalip=`. 

`nat=true` = `lnd` will query your router's upnp implmentation to detect a change in ip, then broadcast the new ip to the network. However, this uses universal plug and play or NAT-PMP, which is a [security vulnerability](https://docs.netgate.com/pfsense/en/latest/book/services/upnp-and-nat-pmp.html) if not implemented properly on your router.  Follow this [optional guide](https://github.com/seth586/guides/blob/master/OpenWRT/upnp_natpmp.md) to set your OpenWRT router up with NAT-PMP in a secure fashion.

Make sure port `9735` is [forwarded in your router](freenas_1_jail_creation.md) to your bitcoin jail!

Save (CTRL+O), then exit (CTRL+X)

### LND Startup and initialization
Start `lnd`:
```
# sudo -u bitcoin /usr/local/bin/lnd --configfile=/usr/local/etc/lnd.conf
```
If it works, you should see the following message:
```
Attempting automatic RPC configuration to bitcoind
Automatically obtained bitcoind's RPC credentials
2019-02-07 22:00:34.994 [INF] LTND: Version: 0.5.2-beta commit=v0.5.2-beta, build=production, logging=default
2019-02-07 22:00:34.994 [INF] LTND: Active chain: Bitcoin (network=mainnet)
2019-02-07 22:00:35.013 [INF] CHDB: Checking for schema update: latest_version=7, db_version=7
2019-02-07 22:00:35.054 [INF] RPCS: password gRPC proxy started at [::]:8082
2019-02-07 22:00:35.054 [INF] RPCS: password RPC server listening on 127.0.0.1:10009
2019-02-07 22:00:35.054 [INF] LTND: Waiting for wallet encryption password. Use `lncli create` to create a wallet, `lncli unlock` to unlock an existing wallet, or `lncli changepassword` to change the password of an existing wallet and unlock it.
```
Open another SSH terminal window, log into to your FreeNAS server, and switch to your bitcoin jail. We will use `lncli` to create a wallet and store the recovery key.
```
# sudo -u bitcoin lncli -lnddir "/var/db/lnd" create
```
Follow the prompt to create a wallet. Write down your 24 word seed on paper, and store it somewhere safe. Pick a strong wallet password, too!

We are done with this terminal, close it.

In your other terminal window, `lnd` will begin its sync. Once the sync is complete, you will see a bunch of "New channel disocvered" nessages, exit `lnd` (CTRL+C). 

### Configure start on boot & restart

We will again use [daemon](https://www.freebsd.org/cgi/man.cgi?query=daemon) to run our `lnd` process at bootup, and restart the process should it fail.

Lets make the [rc.d script](https://www.freebsd.org/doc/en/articles/rc-scripting/):
```
# nano /usr/local/etc/rc.d/lnd
```
Paste the following service script into nano:
```
#!/bin/sh
#
# PROVIDE: lnd
# REQUIRE: bitcoind
# KEYWORD:

. /etc/rc.subr

name="lnd"
rcvar="lnd_enable"

export HOME=/home/bitcoin

lnd_command="/usr/local/bin/lnd --configfile=/usr/local/etc/lnd.conf"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-u bitcoin -P ${pidfile} -r -f ${lnd_command}"

load_rc_config $name
: ${lnd_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O,ENTER) and exit (CTRL+X)

Make the startup script executable:
```
# chmod +x /usr/local/etc/rc.d/lnd
```

Enable our service in `/etc/rc.conf`
```
# nano /etc/rc.conf
```
Append the following line
```
lnd_enable="YES"
```
Save, (CTRL+O,ENTER) then exit (CTRL+O)

Lets verify lnd auto boots on startup:
```
# exit
root@freenas[~]# iocage restart bitcoin
root@freenas[~]# iocage console bitcoin
# ps aux
```

Note: Any time `lnd` reboots, you will need to unlock the wallet again. Open a seperate SSH window, log in to your bitcoin jail, switch to bitcoin user `su bitcoin`, and type `lncli unlock`. Type in the password to unlock your wallet, then you can exit this extra SSH window. This is a security function in case someone steals your server! In the next guide, you will install a web user interface called `RTL`, which makes unlocking your wallet much easier.

### Upgrade LND
Read the release notes, if a lot changed, you may have to close channels or do something to prepare for the upgrade! I'll keep a log of upgrade notes beyond 0.5.2 below:
```
# service lnd stop
# cd ~
# wget https://github.com/lightningnetwork/lnd/releases/download/v0.5.2-beta/lnd-freebsd-amd64-v0.5.2-beta.tar.gz
# tar -xzf lnd-freebsd-amd64-v0.5.2-beta.tar.gz
# cd lnd-freebsd-amd64-v0.5.2-beta
# install -m 0755 -o root -g wheel lnd lncli /usr/local/bin
# cd ~
# rm -r lnd-freebsd-amd64-v0.5.2-beta
# service lnd start
```

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
