[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd ](freenas_5_lnd.md)] - [**loopd**] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & Lightning️ on FreeNAS / TrueNAS

### Install Lightning Lab's Loop client

If not already there, SSH into your freenas box as root, then switch to your bitcoin jail:
```
root@freenas[~] # iocage console bitcoin
```

Check [lightning lab's loop github repo](https://github.com/lightninglabs/loop/releases) for the latest release, make sure you select the correct binaries for your processor and operating system. (amd64 is for amd and intel processors)
```
# cd ~
# wget https://github.com/lightninglabs/loop/releases/download/v0.7.0-beta/loop-freebsd-amd64-v0.7.0-beta.tar.gz
# tar -xvf loop-freebsd-amd64-v0.7.0-beta.tar.gz
# install -m 0755 -o root -g wheel loop-freebsd-amd64-v0.7.0-beta/loop loop-freebsd-amd64-v0.7.0-beta/loopd /usr/local/bin
# rm -r loop-freebsd-amd64-v0.7.0-beta loop-freebsd-amd64-v0.7.0-beta.tar.gz
```

### loopd Startup and initialization
Start `loopd`:
```
# loopd --lnd.macaroondir=/var/db/lnd/data/chain/bitcoin/mainnet --lnd.tlspath=/var/db/lnd/tls.cert --proxy=localhost:9050
```
If it works, you should see the following message:
```
2020-06-09 21:56:54.119 [INF] LOOPD: Version: 0.6.3-beta commit=
2020-06-09 21:56:54.119 [INF] LNDC: Creating lnd connection to localhost:10009
2020-06-09 21:56:54.119 [INF] LNDC: Connected to lnd
2020-06-09 21:56:54.170 [INF] LNDC: Using network mainnet
2020-06-09 21:56:54.170 [INF] LOOPD: Swap server address: swap.lightning.today:11010
2020-06-09 21:56:54.207 [INF] STORE: Checking for schema update: latest_version=3, db_version=3
2020-06-09 21:56:54.207 [INF] LOOP: Proxying connection to swap.lightning.today:11010 over Tor SOCKS proxy localhost:9050
2020-06-09 21:56:54.208 [INF] LOOPD: Starting swap client
2020-06-09 21:56:54.208 [INF] LOOP: Connected to lnd node 'seth586😈guides' with pubkey 023ec3d1fa35f7fb8996374cf1848c1a40788df013551c5510c75617222bd2dd2d (version v0.10.1-beta, build tags 'autopilotrpc,signrpc,walletrpc,chainrpc,invoicesrpc,watchtowerrpc')
2020-06-09 21:56:54.208 [INF] LOOPD: Waiting for updates
2020-06-09 21:56:54.208 [INF] LOOPD: Starting gRPC listener
2020-06-09 21:56:54.208 [INF] LOOPD: Starting REST proxy listener
2020-06-09 21:56:54.208 [INF] LOOPD: REST proxy listening on 127.0.0.1:8081
2020-06-09 21:56:54.208 [INF] LOOPD: RPC server listening on 127.0.0.1:11010
2020-06-09 21:56:54.209 [INF] LOOP: Wait for first block ntfn
2020-06-09 21:56:54.209 [INF] LOOP: Starting event loop at height 633975
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

loopd_command="/usr/local/bin/loopd --lnd.macaroondir=/var/db/lnd/data/chain/bitcoin/mainnet --lnd.tlspath=/var/db/lnd/tls.cert --proxy=localhost:9050"
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
Save, (CTRL+O,ENTER) then exit (CTRL+X)

Now start the service:
```
# service loopd start
```
### loopd liquidity
loopd uses a liquidity provider that you send off-chain funds to receive an on-chain transaction (loop out). Make sure you have an adequate liquidity path (or open a channel directly) with their node:
```
021c97a90a411ff2b10dc2a8e32de2f29d2fa49d41bfbb52bd416e460db0747d0d@18.224.56.146:9735
```

### Upgrade loopd
```
# service loopd stop
# cd ~
# wget https://github.com/lightninglabs/loop/releases/download/v0.7.0-beta/loop-freebsd-amd64-v0.7.0-beta.tar.gz
# tar -xvf loop-freebsd-amd64-v0.7.0-beta.tar.gz
# install -m 0755 -o root -g wheel loop-freebsd-amd64-v0.7.0-beta/loop loop-freebsd-amd64-v0.7.0-beta/loopd /usr/local/bin
# rm -r loop-freebsd-amd64-v0.7.0-beta loop-freebsd-amd64-v0.7.0-beta.tar.gz
# service loopd start && tail -f /root/.loop/logs/mainnet/loopd.log
```

Verify the logs show the service started sucessfully, kill `tail` with CTRL+C

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
