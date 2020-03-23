[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd ](freenas_5_lnd.md)] - [**loopd**] - [ [RTL](freenas_6_rtl.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Install Lightning Lab's Loop service

If not already there, SSH into your freenas box as root, then switch to your bitcoin jail:
```
root@freenas[~] # iocage console bitcoin
```

Check [lightning lab's loop github repo](https://github.com/lightninglabs/loop/releases) for the latest release, make sure you select the correct binaries for your processor and operating system. (amd64 is for amd and intel processors)
```
# cd ~
# wget https://github.com/lightninglabs/loop/releases/download/v0.5.1-beta/loop-freebsd-amd64-v0.5.1-beta.tar.gz
# tar -xvf loop-freebsd-amd64-v0.5.1-beta.tar.gz
# cd loop-freebsd-amd64-v0.5.1-beta
# install -m 0755 -o root -g wheel loop loopd /usr/local/bin
# cd ~
# rm -r loop-freebsd-amd64-v0.5.1-beta
# rm loop-freebsd-amd64-v0.5.1-beta.tar.gz
```


### loopd Startup and initialization
Start `loopd`:
```
# loopd --lnd.macaroondir=/var/db/lnd/data/chain/bitcoin/mainnet --lnd.tlspath=/var/db/lnd/tls.cert
```
If it works, you should see the following message:
```
2020-03-23 13:00:16.634 [INF] LOOPD: Version: 0.5.1-beta commit=
2020-03-23 13:00:16.688 [INF] LNDC: Creating lnd connection to localhost:10009
2020-03-23 13:00:16.830 [INF] LNDC: Connected to lnd
2020-03-23 13:00:17.528 [INF] LNDC: Using network mainnet
2020-03-23 13:00:17.528 [INF] LOOPD: Swap server address: swap.lightning.today:11010
2020-03-23 13:00:17.848 [INF] STORE: Initializing new database with version 3
2020-03-23 13:00:18.105 [INF] STORE: Checking for schema update: latest_version=3, db_version=3
2020-03-23 13:00:18.105 [INF] LOOPD: Starting gRPC listener
2020-03-23 13:00:18.105 [INF] LOOPD: Starting REST proxy listener
2020-03-23 13:00:18.105 [INF] LOOPD: Waiting for updates
2020-03-23 13:00:18.105 [INF] LOOPD: RPC server listening on 127.0.0.1:11010
2020-03-23 13:00:18.106 [INF] LOOPD: REST proxy listening on 127.0.0.1:8081
2020-03-23 13:00:18.105 [INF] LOOPD: Starting swap client
2020-03-23 13:00:18.145 [INF] LOOP: Connected to lnd node seth586😈guides with pubkey 023ec3d1fa35f7fb8996374cf1848c1a40788df013551c5510c75617222bd2dd2d
2020-03-23 13:00:18.145 [INF] LOOP: Wait for first block ntfn
2020-03-23 13:00:18.163 [INF] LOOP: Starting event loop at height 622668
```
Press Ctrl+C to stop the program.

### Configure start on boot & restart

We will again use [daemon](https://www.freebsd.org/cgi/man.cgi?query=daemon) to run our `loopd` process at bootup, and restart the process should it fail.

Lets make the [rc.d script](https://www.freebsd.org/doc/en/articles/rc-scripting/):
```
# nano /usr/local/etc/rc.d/loopd
```
Paste the following service script into nano:
```
#!/bin/sh
#
# PROVIDE: loopd
# REQUIRE: bitcoind tor lnd
# KEYWORD:

. /etc/rc.subr

name="loopd"
rcvar="loopd_enable"

loopd_command="/usr/local/bin/loopd --lnd.macaroondir=/var/db/lnd/data/chain/bitcoin/mainnet --lnd.tlspath=/var/db/lnd/tls.cert"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${loopd_command}"

load_rc_config $name
: ${loopd_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O,ENTER) and exit (CTRL+X)

Make the startup script executable:
```
# chmod +x /usr/local/etc/rc.d/loopd
```

Enable our service in `/etc/rc.conf`
```
# nano /etc/rc.conf
```
Append the following line
```
loopd_enable="YES"
```
Save, (CTRL+O,ENTER) then exit (CTRL+O)

Now start the service:
```
# service loopd start
```

### Upgrade loopd
Read the release notes, if a lot changed, you may have to close channels or do something to prepare for the upgrade! I'll keep a log of upgrade notes beyond 0.5.2 if anything breaks by upgrading below:
(0.7.1 -> 0.8.0 - the channel databse needs to migrate. Make sure this process is sucessful, if not, revert to 0.7.1)
(0.9.0 also needs a database migration, verify it runs sucessfully as described below)
```
# service lnd stop
# cd ~
# wget https://github.com/lightninglabs/loop/releases/download/v0.5.1-beta/loop-freebsd-amd64-v0.5.1-beta.tar.gz
# tar -xvf loop-freebsd-amd64-v0.5.1-beta.tar.gz
# cd loop-freebsd-amd64-v0.5.1-beta
# install -m 0755 -o root -g wheel lnd lncli /usr/local/bin
# cd ~
# rm -r loop-freebsd-amd64-v0.5.1-beta
# rm loop-freebsd-amd64-v0.5.1-beta.tar.gz
# lnd --configfile=/usr/local/etc/lnd.conf
```
Save (Ctrl+O & ENTER) and exit (Ctrl+X)

```
# service loopd start
```

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
