WORK IN PROGRESS, DO NOT USE

1. Create Jail

2. Change repository to latest
```
# mkdir /usr/local/etc/pkg
# mkdir /usr/local/etc/pkg/repos
# echo 'FreeBSD: { url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest" }' >> /usr/local/etc/pkg/repos/FreeBSD.conf
```

3. Install `bitcoin-daemon`
```
# pkg install -y bitcoin-daemon bitcoin-utils
# sysrc bitcoind_enable="YES"
# echo 'server=1' >> /usr/local/etc/bitcoin.conf
# echo 'txindex=1' >> /usr/local/etc/bitcoin.conf
# echo 'zmqpubrawblock=tcp://127.0.0.1:28332' >> /usr/local/etc/bitcoin.conf
# echo 'zmqpubrawtx=tcp://127.0.0.1:28333' >> /usr/local/etc/bitcoin.conf
# service bitcoind start
```

4. Wait until sync is complete, once blocks=headers you're good to go. Let this run overnight.
```
# bitcoin-cli --datadir=/var/db/bitcoin getblockchaininfo
```

5. Install c-lightning:
```
# pkg install -y autoconf automake git gmp asciidoc gmake libtool python python3 sqlite3 libsodium
# cd ~
# git clone https://github.com/ElementsProject/lightning.git
# cd lightning
# ./configure
# gmake
# gmake install
# nano /usr/local/etc/rc.d/lightningd
```
paste the following:
```
#!/bin/sh
#
# PROVIDE: lightningd
# REQUIRE: bitcoind
# KEYWORD:

. /etc/rc.subr

name="lightningd"
rcvar="lightningd_enable"
lightningd_command="/usr/local/bin/lightningd /usr/local/etc/lightningd.conf"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u bitcoin -r -f ${lightningd_command}"

load_rc_config $name
: ${lightningd_enable:=no}

run_rc_command "$1"
```
