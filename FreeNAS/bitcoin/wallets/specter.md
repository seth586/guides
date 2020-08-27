[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

## Specter Daemon
[Specter](https://github.com/cryptoadvance/specter-desktop) is an electrum alternative that connects directly to bitcoin core. By hosting the daemon version on freenas, you can remotely generate, sign with coldcard, and broadcast PSBT transactions without having to install client side software. 

If you want USB connectivity for a trezor or ledger type of device, you will need to run specter desktop on the client machine as well to create the USB bridge.

Minimizing the amount of software that has file access to your lightning hot wallet is considered a security best practice, so lets [spin up a new jail](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/freenas_1_jail_creation.md). I will name this jail `specter` and assign it an ip address of `192.168.84.11`. SSH in and lets begin!

```
root@freenas:~ # iocage console specter
root@specter:~ # pkg install python3 py37-pip nano
root@specter:~ # fetch https://github.com/cryptoadvance/specter-desktop/archive/v0.7.1.tar.gz
root@specter:~ # tar -xvf v0.7.1.tar.gz
root@specter:~ # cd specter-desktop-0.7.1
root@specter:~/specter-desktop-0.7.1 # pip-3.7 install -e .
root@specter:~/specter-desktop-0.7.1 # cd ~
root@specter:~ #
root@specter:~/specter-desktop-0.7.1 # python3 -m cryptoadvance.specter server --host 0.0.0.0
```

Lets create a service, `mkdir /usr/local/etc/rc.d && nano /usr/local/etc/rc.d/specter`:
```
#!/bin/sh
#
# PROVIDE: specter
# REQUIRE: bitcoind tor
# KEYWORD:

. /etc/rc.subr

name="specter"
rcvar="specter_enable"

specter_command="/usr/local/bin/python3 -m cryptoadvance.specter server --host 0.0.0.0"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${lnd_command}"

load_rc_config $name
: ${lnd_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O, ENTER) and exit (CTRL+X). Make the rc.d script executable and enable our service 

```
root@specter:~ # chmod +x /usr/local/etc/rc.d/specter
root@specter:~ # sysrc specter_enable="YES"
```

