[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] -  **[Extras]** 

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### My Extras

#### [Run LND on clearnet](https://github.com/seth586/guides/blob/master/OpenWRT/upnp_natpmp.md)
Don't have a static IP address? `lnd` will loose peer connections unless you configure `nat=true` and have a working UPnP implementation. This is a process to ensure your UPnP implementation is secure.

#### [Manually compile bitcoind](extras/compile_bitcoind.md)
Some situations require you to compile directly from source code.  

#### [Electrs: Electrum Server in Rust](extras/electrs.md)
This is a fully indexed Electrum Server. It adds +40GB or so, but can look up any address or xpub, whereas Electrum Personal Server is lightweight, but only indexes addresses and xpubs you specify.

#### [Tor relay node](extras/torrelay.md) 
Tor doesn't give anonymity or sufficient bandwidth unless there are enough volunteer relays helping the network. Lets give back!

### Mobile Wallets for Android over Tor Hidden Service
Connect securely, privately, and anonymously to your home node wherever you are in the world!


#### [Zeus LN](wallets/zeusln.md)

#### [Blockstream Green](wallets/green.md) 

#### [BlueWallet](wallets/bluewallet.md)

#### [Zap for Android](wallets/zapandroid.md)

### Mobile Wallets for iOS over Tor Hidden Service

#### [Zap iOS](wallets/zap.md)

### External guides

### Browser Enabled Wallets over Tor Hidden Service

#### [ [Joule](freenas_7_joule.md) ]


#### [SoftEther VPN in a FreeNAS jail](https://forums.freenas.org/index.php?threads/alternative-to-openvpn-softether-vpn.47395/)
Want to use a mobile lightning wallet away from your home network? Set up a VPN server at home, so you can securely connect to your `lnd` on the road! SoftEther offers a free DNS service, great option if your ISP changes your home IP address on you! 
