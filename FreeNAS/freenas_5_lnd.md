[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [**lnd**] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Install Lightning Lab's LND

If not already there, SSH into your freenas box as root, then switch to your bitcoin jail:
```
root@freenas[~] # iocage console bitcoin
```

Check [LND's github repo](https://github.com/lightningnetwork/lnd/releases) for the latest release, make sure you select the correct binaries for your processor and operating system. (amd64 is for amd and intel processors)
```
# pkg install wget ca_root_nss
# cd ~
# wget https://github.com/lightningnetwork/lnd/releases/download/v0.10.0-beta/lnd-freebsd-amd64-v0.10.0-beta.tar.gz
# tar -xvf lnd-freebsd-amd64-v0.10.0-beta.tar.gz
# cd lnd-freebsd-amd64-v0.10.0-beta
# install -m 0755 -o root -g wheel lnd lncli /usr/local/bin
# cd ~
# rm -r lnd-freebsd-amd64-v0.10.0-beta
# rm lnd-freebsd-amd64-v0.10.0-beta.tar.gz
# nano /usr/local/etc/lnd.conf
```
### LND Configuration
Read up on configuration options [here](https://github.com/lightningnetwork/lnd/blob/master/sample-lnd.conf). 
Add the following lines to your `lnd.conf` file:
```
[Application Options]
lnddir=/var/db/lnd
alias=insert_something_catchy_here
listen=localhost
restlisten=127.0.0.1:8080
rpclisten=127.0.0.1:10009
tlsextraip=0.0.0.0
minchansize=900000
maxlogfiles=1
maxlogfilesize=10
accept-keysend=1

[Bitcoin]
bitcoin.active=1
bitcoin.mainnet=1
bitcoin.node=bitcoind
bitcoin.basefee=1000
bitcoin.feerate=100
bitcoin.timelockdelta=40

[Bitcoind]
bitcoind.dir=/var/db/bitcoin

[tor]
tor.active=1
tor.socks=localhost:9050
tor.dns=soa.nodes.lightning.directory:53
tor.control=localhost:9051
tor.v3=1
```
### Configuration Notes
This configuration uses tor for NAT traversal and to prevent doxing your home IP address. Don't tell the world "this house has bitcoins!"! If you want to run on clearnet and advertise your home IP address, check out the [Extras](extras.md) page to set up `nat=true` in a secure fashion.

Fees. You may have to pay fees to other nodes when you rebalance channels, and you may have to close and reopen channels to disconected nodes, which will require on-chain fees. Don't operate at a loss! Do NOT make a 0 fee node, this will leave you vulnerable to denial of service attacks!

`bitcoin.basefee=1000` = Fee of 1 satoshi per payment forwarded

`bitcoin.feerate=100` = Fee of 100 satoshis per million forwarded (0.01% fee)

Save (CTRL+O), then exit (CTRL+X)

### LND Startup and initialization
Start `lnd`:
```
# lnd --configfile=/usr/local/etc/lnd.conf
```
If it works, you should see the following message:
```
Attempting automatic RPC configuration to bitcoind
Automatically obtained bitcoind's RPC credentials
2019-02-07 22:00:34.994 [INF] LTND: Version: 0.5.2-beta commit=v0.5.2-beta, build=production, logging=default
2019-02-07 22:00:34.994 [INF] LTND: Active chain: Bitcoin (network=mainnet)
2019-02-07 22:00:35.013 [INF] CHDB: Checking for schema update: latest_version=7, db_version=7
2019-02-07 22:00:35.054 [INF] RPCS: password gRPC proxy started at [::]:8080
2019-02-07 22:00:35.054 [INF] RPCS: password RPC server listening on 127.0.0.1:10009
2019-02-07 22:00:35.054 [INF] LTND: Waiting for wallet encryption password. Use `lncli create` to create a wallet, `lncli unlock` to unlock an existing wallet, or `lncli changepassword` to change the password of an existing wallet and unlock it.
```
Open another SSH terminal window, log into to your FreeNAS server, and switch to your bitcoin jail. We will use `lncli` to create a wallet and store the recovery key.
```
# lncli -lnddir "/var/db/lnd" create
```
Follow the prompt to create a wallet. Pick a strong wallet password. Write down your 24 word seed on paper, and store it somewhere safe!

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
# REQUIRE: bitcoind tor
# KEYWORD:

. /etc/rc.subr

name="lnd"
rcvar="lnd_enable"

lnd_command="/usr/local/bin/lnd --configfile=/usr/local/etc/lnd.conf"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${lnd_command}"

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
### Unlock wallet on `lnd` reboot or restart
Note: Any time `lnd` reboots, you will need to unlock the wallet again.  
```
# lncli -lnddir "/var/db/lnd" unlock
```
Type in the password to unlock your wallet. This is a security function in case someone steals your server! In the next guide, you will install a web user interface called `RTL`, which makes unlocking your wallet much easier.

### Upgrade LND
Read the release notes, if a lot changed, you may have to close channels or do something to prepare for the upgrade!
```
# service lnd stop
# cd ~
# wget https://github.com/lightningnetwork/lnd/releases/download/v0.10.1-beta.rc2/lnd-freebsd-amd64-v0.10.1-beta.rc3.tar.gz
# tar -xvf lnd-freebsd-amd64-v0.10.1-beta.rc3.tar.gz
# install -m 0755 -o root -g wheel ~/lnd-freebsd-amd64-v0.10.1-beta.rc3/lnd ~/lnd-freebsd-amd64-v0.10.1-beta.rc3/lncli /usr/local/bin
# rm -r lnd-freebsd-amd64-v0.10.1-beta.rc3
# rm lnd-freebsd-amd64-v0.10.1-beta.rc3.tar.gz
# lnd --configfile=/usr/local/etc/lnd.conf
```

Watch the console to make sure that the database migration is sucessful. Ctrl+C to shut down lnd, then start the service:

```
# service lnd start
```

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
