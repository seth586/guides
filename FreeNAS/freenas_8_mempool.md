[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - **[mempool]** - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

## mempool

When transacting on chain, you need a good understanding on what network fees are currently trending so you can set yuour transaction priority accordingly. The best visual tool I've ever found is this gem called [mempool](https://github.com/mempool/mempool). 

## Create RPC credentials for Bitcoin Core

SSH into your bitcoin jail.

```
# iocage console bitcoin
```

Download the rpcauth tool as documented [here](https://github.com/bitcoin/bitcoin/tree/master/share/rpcauth).

```
# fetch https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py
# python3 ./rpcauth.py mempool
String to be appended to bitcoin.conf:
rpcauth=mempool:5d0d70936350d0a79b588a9bb2906ea1$82afc2d29dfcfd808acd98f855cf47989564d8f1cd55b515f23fb10ace0dd75a
Your password:
2tm5NiN8wZVyjx_hgUL5O8it68WfoadHDEZ-v6w_RhQ=
```

Save this information. Add the `rpcauth=` string above to `bitcoin.conf`:
```
# nano /usr/local/etc/bitcoin.conf
```
Save (CTRL+O,ENTER) and exit (CTRL+X)

