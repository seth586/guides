[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor & i2p](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - **[Extras]** 

## TrueNASnode - full bitcoin stack deployment guide ![BSDBTC60.png](images/BSDBTC60.png)

Join the chatroom on the matrix chat protocol: [#truenasnode:nym.im](https://matrix.to/#/#truenasnode:nym.im)

### My Extras

#### [Run LND on clearnet](https://github.com/seth586/guides/blob/master/OpenWRT/upnp_natpmp.md)
Don't have a static IP address? `lnd` will loose peer connections unless you configure `nat=true` and have a working UPnP implementation. This is a process to ensure your UPnP implementation is secure.

#### [Manually compile bitcoind](extras/compile_bitcoind.md)
Some situations require you to compile directly from source code.  

### Mobile Wallets for Android over Tor Hidden Service
Connect securely, privately, and anonymously to your home node wherever you are in the world!


#### [Zeus LN](wallets/zeusln.md)

#### [Blockstream Green](wallets/green.md) 

#### [BlueWallet](wallets/bluewallet.md)

### Mobile Wallets for iOS over Tor Hidden Service

#### [Zap iOS](wallets/zap.md)

### Browser Enabled Wallets

#### [Joule](wallets/joule.md)

#### [Specter](wallets/specter.md) - A new electrum alternative

### External guides

#### [SoftEther VPN in a FreeNAS jail](https://forums.freenas.org/index.php?threads/alternative-to-openvpn-softether-vpn.47395/)
Want to use a mobile lightning wallet away from your home network? Set up a VPN server at home, so you can securely connect to your `lnd` on the road! SoftEther offers a free DNS service, great option if your ISP changes your home IP address on you! 
