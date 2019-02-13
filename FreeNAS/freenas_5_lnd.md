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
Make sure to set `externalip=` if you have a static ip. If your ISP assigns a dynamic ip address, `nat=true` can be used instead. However, this uses universal plug and play or NAT-PMP, which is a [security vulnerability](https://docs.netgate.com/pfsense/en/latest/book/services/upnp-and-nat-pmp.html) if not implemented correctly. Incoming channels will not be able to connect if your advertised IP address is not correct. Follow this [optional guide](https://github.com/seth586/guides/blob/master/OpenWRT/upnp_natpmp.md) to set your OpenWRT router up with NAT-PMP.

Make sure port `9735` is [forwarded in your router](freenas_1_jail_creation.md) to your bitcoin jail!

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

We are done with this terminal, close it.

In your other terminal window, `lnd` will begin its sync. Once the sync is complete, you will see a bunch of "New channel disocvered" nessages, exit `lnd` (CTRL+C). Exit the bitcoin user:
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

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
