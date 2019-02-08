[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [**lnd**] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Install Lightning Lab's LND

#### 🚧🚧🚧THIS SECTION IS STILL UNDER CONSTRUCTION, DO NOT USE!🚧🚧🚧

Note: Lightning is developing at a rapid pace, and install changes may occur between versions. This section will provide install & update instructions for each version, so pay attention to what part of the guide to use.

If not already there, SSH into your freenas box as root, then switch to your bitcoin jail:
```
root@freenas[~] # iocage console bitcoin
```

Install prerequisites for LND 0.5.1:
```
# pkg install go git
```

Set environment variables for go:
```
# cd ~
# nano .cshrc
```
Add the following lines to the bottom of the file, 
```
setenv GOPATH /root/.gopkg
set path = ($path /usr/local/go/bin /root/.gopkg/bin)
```
Save (CTRL+O, ENTER) then exit (CTRL+X)
