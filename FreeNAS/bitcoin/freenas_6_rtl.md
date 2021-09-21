[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor & i2p](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [**RTL**] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ]

## TrueNASnode - full bitcoin stack deployment guide ![BSDBTC60.png](images/BSDBTC60.png)

Running `lnd' from the command line is exhausting, lets get a pretty user interface going!

### Install RTL
Read up on RTL [here](https://github.com/ShahanaFarooqui/RTL). Find the latest RTL release [here](https://github.com/ShahanaFarooqui/RTL/releases)

If not already there, SSH into your freenas box and switch to your bitcoin jail.

```
# pkg install node npm python cairo
# cd ~
# wget https://github.com/Ride-The-Lightning/RTL/archive/refs/tags/v0.11.0.tar.gz
# tar -xvf v0.11.0.tar.gz
# rm v0.11.0.tar.gz
# mv ~/RTL-0.11.0 ~/rtl
# cd rtl
# npm install --only=production
```
Once the install is complete, create RTL-Config.json [configuration options](https://github.com/Ride-The-Lightning/RTL/blob/master/docs/Application_configurations):
```
# nano ~/rtl/RTL-Config.json
```
Edit the following lines, make sure to set `rtlPass=`:
```
{
  "multiPass": "enteryourownpasswordhere",
  "port": "3000",
  "defaultNodeIndex": 1,
  "SSO": {
    "rtlSSO": 0,
    "rtlCookiePath": "",
    "logoutRedirectLink": ""
  },
  "nodes": [
    {
      "index": 1,
      "lnNode": "My BSD Node",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/var/db/lnd/data/chain/bitcoin/mainnet",
        "configPath": "/usr/local/etc/lnd.conf"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "NIGHT",
        "themeColor": "PURPLE",
        "currencyUnit": "USD",
        "bitcoindConfigPath": "/usr/local/etc/bitcoin.conf",
        "channelBackupPath": "/root/rtl",
        "enableLogging": false,
        "lnServerUrl": "https://localhost:8080/v1",
        "swapServerUrl": "https://localhost:8081/v1",
        "fiatConversion": false
      }
    }
  ]
}


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
# REQUIRE: bitcoind lnd loopd
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

### Set loop macaroon path in RTL

In the RTL web UI go to: Node Config > Services > Loop
Enter Loop Macaroon Path 
```
/root/.loop/mainnet
```
Click Update

### Upgrade RTL

```
# service rtl stop
# wget https://github.com/Ride-The-Lightning/RTL/archive/refs/tags/v0.11.0.tar.gz
# tar -xvf v0.11.0.tar.gz
# cp ~/rtl/RTL-Config.json ~/RTL-0.11.0/RTL-Config.json
# rm -r ~/rtl
# mv ~/RTL-0.11.0 ~/rtl
# rm v0.11.0.tar.gz
# cd rtl
# npm install --only=production
# service rtl start
```


Next: [ [mempool](freenas_8_mempool.md) ]
