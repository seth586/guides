[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor & i2p](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd ](freenas_5_lnd.md)] - [**loopd**] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ]

## TrueNASnode - full bitcoin stack deployment guide ![BSDBTC60.png](images/BSDBTC60.png)

Join the chatroom on the matrix chat protocol: [#truenasnode:nym.im](https://matrix.to/#/#truenasnode:nym.im)

### Install Lightning Lab's Loop client

If not already there, SSH into your freenas box as root, then switch to your bitcoin jail:
```
root@freenas[~] # iocage console bitcoin
```

Check [lightning lab's loop github repo](https://github.com/lightninglabs/loop/releases) for the latest release, make sure you select the correct binaries for your processor and operating system. (amd64 is for amd and intel processors)
```
# cd ~
# wget https://github.com/lightninglabs/loop/releases/download/v0.23.0-beta/loop-freebsd-amd64-v0.23.0-beta.tar.gz
# tar -xvf loop-freebsd-amd64*
# install -m 0755 -o root -g wheel loop-freebsd-amd64*/* /usr/local/bin
# rm -r /loop-freebsd-amd64* loop-freebsd-amd64*
```

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

loopd_command="/usr/local/bin/loopd --configfile=/usr/local/etc/loopd.conf"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u lnd -r -f ${loopd_command}"

load_rc_config $name
: ${loopd_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O,ENTER) and exit (CTRL+X)

Make the startup script executable:
```
# chmod +x /usr/local/etc/rc.d/loopd
```

Enable our service with `nano /etc/rc.conf` and appent the following line:
```
loopd_enable="YES"
```
Save, (CTRL+O,ENTER) then exit (CTRL+X)

Set permissions and create config file:
```
mkdir /var/db/loopd && chown lnd:lnd /var/db/loopd
cd /usr/local/etc
touch loopd.conf && chown lnd:lnd loopd.conf && chmod 600 loopd.conf && nano loopd.conf
```
```
[Application Options]
loopdir=/var/db/loopd
configfile=/usr/local/etc/loopd.conf

[lnd]
lnd.macaroonpath=/var/db/lnd/data/chain/bitcoin/mainnet/admin.macaroon
lnd.tlspath=/var/db/lnd/tls.cert

[server]
server.proxy=localhost:9050
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

Now start the service:
```
# service loopd start && tail -f /var/db/loopd/logs/mainnet/loopd.log
```
Logs should appear normal, press CTRL+C to stop following the logs

### loopd liquidity
loopd uses a liquidity provider that you send off-chain funds to receive an on-chain transaction (loop out). Make sure you have an adequate liquidity path (or open a channel directly) with their node:
```
021c97a90a411ff2b10dc2a8e32de2f29d2fa49d41bfbb52bd416e460db0747d0d@18.224.56.146:9735
```

### Upgrade loopd
```
# service loopd stop
# cd ~
# wget https://github.com/lightninglabs/loop/releases/download/v0.23.0-beta/loop-freebsd-amd64-v0.23.0-beta.tar.gz
# tar -xvf loop-freebsd-amd64*
# install -m 0755 -o root -g wheel loop-freebsd-amd64*/* /usr/local/bin
# rm -r /loop-freebsd-amd64* loop-freebsd-amd64*
# service loopd start && tail -f /var/db/loopd/logs/mainnet/loopd.log
```

Verify the logs show the service started sucessfully, kill `tail` with CTRL+C

Next: { [Install Ride The Lightning web UI](freenas_6_rtl.md) ]
