[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [**RTL**] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

#### 🚧🚧🚧THIS SECTION IS STILL UNDER CONSTRUCTION, DO NOT USE!🚧🚧🚧

Running `lnd' from the command line is exhausting, lets get a pretty user interface going!

### Install RTL
Find the latest release [here](https://github.com/ShahanaFarooqui/RTL/releases).

If not already there, SSH into your freenas box and switch to your bitcoin jail.

```
# pkg install node npm python
# cd ~
# git clone https://github.com/ShahanaFarooqui/RTL.git
# cd RTL
# npm install

