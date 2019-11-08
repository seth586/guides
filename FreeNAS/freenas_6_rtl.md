[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [**RTL**] - [ [Joule](freenas_7_joule.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

Running `lnd' from the command line is exhausting, lets get a pretty user interface going!

### Install RTL
Read up on RTL [here](https://github.com/ShahanaFarooqui/RTL). Find the latest RTL release [here](https://github.com/ShahanaFarooqui/RTL/releases)

If not already there, SSH into your freenas box and switch to your bitcoin jail.

```
# pkg install node npm python cairo
# cd ~
# wget https://github.com/ShahanaFarooqui/RTL/archive/v0.5.2.tar.gz
# tar -xvf v0.5.2.tar.gz
# rm v0.5.2.tar.gz
# mv ~/RTL-0.5.2 ~/rtl
# cd rtl
# npm install
```
Once the install is complete, create RTL.conf [configuration options](https://github.com/ShahanaFarooqui/RTL/blob/master/Application_configurations):
```
# nano RTL.conf
```
Add the following lines, make sure to set `rtlPass=`:
```
[Authentication]
macaroonPath=/var/db/lnd/data/chain/bitcoin/mainnet
nodeAuthType=CUSTOM
lndConfigPath=/usr/local/etc/lnd.conf
rtlPass=password

[Settings]
flgSidenavOpened=true
flgSidenavPinned=true
menu=Vertical
menuType=Regular
theme=dark-blue
satsToBTC=false
lndServerUrl=https://localhost:8080/v1
enableLogging=false
port=3000
bitcoindConfigPath=/usr/local/etc/bitcoin.conf

[SSO]
rtlSSO=0
rtlCookiePath=~/rtl/cookies/auth.cookie
logoutRedirectLink=/login

```
Save (CTRL+O,ENTER) then exit (CTRL+X)

### Configure Autostart
```
# nano /usr/local/etc/rc.d/rtl
```
Add the following lines:
```
#!/bin/sh
#
# PROVIDE: rtl
# REQUIRE: bitcoind lnd
# KEYWORD:

. /etc/rc.subr
name="rtl"
rcvar="rtl_enable"
rtl_command="/usr/local/bin/node /root/rtl/rtl"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${rtl_command}"

load_rc_config $name
: ${rtl_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O,ENTER) then exit (CTRL+X)

Make the service script executable:
```
# chmod +x /usr/local/etc/rc.d/rtl
```
Enable in `etc/rc.conf'
```
# nano /etc/rc.conf
```
Append the following line:
```
rtl_enable="YES"
```
Save (CTRL+O,ENTER) then exit (CTRL+X)

Give it a run!
```
# service rtl start
```

Now connect on your web browser at the jail ip:3000 or myprivateonionaddressocyn4rixm632jid.onion:3000 for remote connections.

### Fund your wallet
LND has its own layer 1 wallet, Click `LND Wallet`, select an `Address Type`, and click `Generate Address`. This wallet will need funds sent to it to create lightning channels. Wait for your transaction to confirm...

### Add a Peer
This step does not commit funds, it just makes a conneciton to the network.

Click on `Peers`, then type an address under `Add Peer`. You can select a peer on www.1ML.com, or connect to my node:
```
023ec3d1fa35f7fb8996374cf1848c1a40788df013551c5510c75617222bd2dd2d@sethnetppzp4diqkclzql5bzsb2rtug74q7yfkuxfcbkpjklniieltid.onion:9735
```

### Add a Channel
Click on `Channels`, select your peer's `Alias`, and select how many satoshis you want to commit. Click `Open`. Wait for your transaction to confirm on the blockchain.

### Managing Channels & Check Balances
In RTL's web-ui, click on channels. All of your inbound and outbound connections are listed here. The most you can receive is under the `Remote Balance` column. The most you can send is under the `Local Balance`. So after making your first channel, you can send, but not receive.

If you make a channel for 100,000 satoshis, and spend 40,000 satoshies, that channel now has a `Local Balance` (spending) of 60,000 and `Remote Balance` of 40,000 (receiving). Total `Local Balance` + `Remote Balance` capacity can not exceed the channel `Capacity`.

Right now it is not possible to send or receive a payment over multiple channels. This future feature is called [Atomic Multipath Payments](https://medium.com/coinmonks/specific-fee-routing-for-multi-path-payments-in-lightning-networks-b0e662c79819), which isn't implemented, yet.

### Spend Some Satoshis!
So go ahead and spend some satoshis! Copy a payment request from a website or service. In RTL's web-ui, click `payments`, and paste lightning invoices in, and click `send payment`.

Give https://www.lightningspin.com/ a spin (its really slick after you install the Joule plugin), give someone a tip on https://tippin.me, read a https://yalls.org/ article. Donate to the [@BTCPayServer fund](https://t.co/FbCF1rtPvY)! Enjoy instantaneous and off chain transactions!

Send me an unreasonably small tip! 
https://tippin.me/@seth586

If you make a channel to me, [contact me](README.md) and I will make a channel back, so you have outbound and inbound capacity!

Over time it is likely you will get inbound connections. Make sure your node is always online, and wallet unlocked.

### Upgrade RTL

```
# service rtl stop
# wget https://github.com/ShahanaFarooqui/RTL/archive/v0.5.3.tar.gz
# tar -xvf v0.5.3.tar.gz
# cp ~/rtl/RTL.conf ~/RTL-0.5.3/RTL.conf
# rm -r ~/rtl
# mv ~/RTL-0.5.3 ~/rtl
# rm v0.5.3.tar.gz
# cd rtl
# npm install
# service rtl start
```


Next: [ [Joule](freenas_7_joule.md) ]
