[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [**lnd**] - [ [RTL](freenas_6_rtl.md) ] - [ [Extras](extras.md) ]

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


### LND Startup and initialization
Start `loopd`:
```
# loopd --lnd.macaroondir=/var/db/lnd/data/chain/bitcoin/mainnet --lnd.tlspath=/var/db/lnd/
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
# PROVIDE: loopd
# REQUIRE: bitcoind tor lnd
# KEYWORD:

. /etc/rc.subr

name="loopd"
rcvar="loopd_enable"

lnd_command="/usr/local/bin/loopd --lnd.macaroondir=/var/db/lnd/data/chain/bitcoin/mainnet --lnd.tlspath=/var/db/lnd/"
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
# wget https://github.com/lightningnetwork/lnd/releases/download/v0.9.1-beta/lnd-freebsd-amd64-v0.9.1-beta.tar.gz
# tar -xvf lnd-freebsd-amd64-v0.9.1-beta.tar.gz
# cd lnd-freebsd-amd64-v0.9.1-beta
# install -m 0755 -o root -g wheel lnd lncli /usr/local/bin
# cd ~
# rm -r lnd-freebsd-amd64-v0.9.1-beta
# rm lnd-freebsd-amd64-v0.9.1-beta.tar.gz
# lnd --configfile=/usr/local/etc/lnd.conf
```

Watch the console to make sure that the database migration is sucessful. If migration is unsucessful, see [this issue](https://github.com/lightningnetwork/lnd/issues/3606). Ctrl+C to shut down lnd, then start the service:

```
# service lnd start
```

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
