[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [**RTL**] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

#### 🚧🚧🚧THIS SECTION IS STILL UNDER CONSTRUCTION, DO NOT USE!🚧🚧🚧

Running `lnd' from the command line is exhausting, lets get a pretty user interface going!

### Install RTL
Find the latest release [here](https://github.com/ShahanaFarooqui/RTL/releases).

If not already there, SSH into your freenas box and switch to your bitcoin jail.

```
# pkg install node npm python cairo
# cd /home/bitcoin
# git clone https://github.com/ShahanaFarooqui/RTL.git
# cd RTL
# npm install
```
Once the install is complete, create a RTL.conf configuration file:
```
# nano RTL.conf
```
Add the following lines:
```
[Authentication]
lndServerUrl=https://localhost:8082/v1
macroonPath=/home/bitcoin/.lnd/data/chain/bitcoin/mainnet
nodeAuthType=CUSTOM
rtlPass=pickapassword
lndConfigPath=/home/bitcoin/.lnd/lnd.conf
[Settings]
flgSidenavOpened=true
flgSidenavPinned=true
menu=Vertical
menuType=Regular
theme=dark-blue
satsToBTC=false
```
Save (CTRL+O,ENTER) then exit (CTRL+X)

### Configure Autostart
```
# nano /usr/local/etc/rc.d/RTL
```
Add the following lines:
```
#!/bin/sh
#
# PROVIDE: RTL
# REQUIRE: bitcoind lnd
# KEYWORD:

. /etc/rc.subr
name="RTL"
rcvar="RTL_enable"
RTL_command="/usr/local/bin/node /home/bitcoin/RTL/rtl"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-u bitcoin -P ${pidfile} -r -f ${RTL_command}"

load_rc_config $name
: ${lnd_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O,ENTER) then exit (CTRL+X)

Make the service script executable:
```
# chmod +x /usr/local/etc/rc.d/RTL
```
Enable in `etc/rc.conf'
```
# nano /etc/rc.conf
```
Append the following line:
```
service RTL_enable="YES"
```
Save (CTRL+O,ENTER) then exit (CTRL+X)

Give it a run!
```
# service RTL start
```

Now connect on your web browser at the jail ip:3000

Its been my experience that RTL will hang with the spinning animation, jsut refresh the page and it should go away.
