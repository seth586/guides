[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [**Bitcoin**] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

View the [Extras](extras.md) page if you want to manually compile bitcoind.

### Bitcoin Install
Secure Socket Shell into your freenas server. SSH is a way to manage your server remotely over a network. When you don’t plug in a monitor & keyboard directly into the server, it’s called a ‘headless’ server. The most popular SSH client is called [PuTTY, download it here](https://www.putty.org/). Connect to your FreeNAS’ IP address, and log in with your root credentials.

```
# iocage list
```

You should see your bitcoin jail listed. Lets switch our console from our base system to our jail.

```
# iocage console bitcoin
```

You're in! Install `bitcoin-daemon`
```
# pkg install -y bitcoin-daemon bitcoin-utils
# sysrc bitcoind_enable="YES"
# cat <<EOT >> /usr/local/etc/bitcoin.conf
server=1
txindex=1
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28333
EOT
# service bitcoind start
# bitcoin-cli --datadir=/var/db/bitcoin getblockchaininfo
```

Wait until sync is complete, once blocks=headers you're good to go. Let this run overnight.

To upgrade bitcoind to a newer available version the easy way, simply type `pkg update && pkg upgrade -y`

Next: [ [Install Tor](freenas_3_tor.md) ]
