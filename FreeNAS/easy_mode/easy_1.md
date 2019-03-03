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
# pkg install bitcoin-daemon bitcoin-utils
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

5. 
