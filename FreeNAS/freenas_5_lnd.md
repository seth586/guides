[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [**lnd**] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

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
# mkdir /home/bitcoin/.lnd
# nano /home/bitcoin/.lnd/lnd.conf
```
### LND Configuration
Read up on configuration options [here](https://github.com/lightningnetwork/lnd/blob/master/sample-lnd.conf)
Add the following lines to your `lnd.conf` file:
```
[Application Options]
externalip=xxx.xxx.xxx.xxx
listen=0.0.0.0:9735
restlisten=0.0.0.0:8082
alias=thisisthenamethatblockexplorerswillshow
maxlogfiles=1
maxlogfilesize=10

[Bitcoin]
bitcoin.active=1
bitcoin.mainnet=1
bitcoin.node=bitcoind

[Bitcoind]
bitcoind.dir=/home/bitcoin/.bitcoin
```
Make sure to set `externalip=` if you have a static ip or dynamic dns address. If your ISP assigns a dynamic ip address, you can still use lightning to send funds, but you will be unable to receive or route payments.

Make sure port `9735` is forwarded in your router to your bitcoin jail!

Save (CTRL+O), then exit (CTRL+X)

### LND Startup and initialization
```
# su bitcoin
bitcoin@bitcoin:~ % lnd
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
root@freenas:~# iocage console bitcoin
# su bitcoin
bitcoin@bitcoin:~ % lncli create
```
Follow the prompt to create a wallet. Pick a strong wallet password!

Were done with this terminal, close it.

In your other terminal window, `lnd` will begin its sync. Once the sync is complete, you will see a bunch of "New channel disocvered" nessages, exit (CTRL+C). Exit the bitcoin user:
```
bitcoin@bitcoin:~ % exit
root@bitcoin
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
# PROVIDE: lnd
# REQUIRE: bitcoind
# KEYWORD:

. /etc/rc.subr

name="lnd"
rcvar="lnd_enable"

export HOME=/home/bitcoin

lnd_command="/usr/local/bin/lnd"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-u bitcoin -P ${pidfile} -r -f ${lnd_command}"

load_rc_config $name
: ${lnd_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O,ENTER) and exit (CTRL+X)

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

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
