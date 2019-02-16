[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ] -  **[Extras]** 

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### My Extras

#### [Configure OpenWRT with UPNP for 'lnd' auto update external ip address](https://github.com/seth586/guides/blob/master/OpenWRT/upnp_natpmp.md)
Don't have a static IP address? `lnd` will loose peer connections unless you configure `nat=true` and have a working UPnP implementation. This is a process to ensure your UPnP implementation is secure.

### Mobile Wallet Guides

#### [Shango - Android / iOS](wallets/shango.md)

### External VPN guides
Want to use a lightning wallet, like [Shango](http://www.shangoapp.com/) on the go? Here are a few options on setting up a VPN server at home, so you can securely connect to your `lnd` on the road!

#### [SoftEther VPN in a FreeNAS jail](https://forums.freenas.org/index.php?threads/alternative-to-openvpn-softether-vpn.47395/)
SoftEther offers a free DNS service, great option if your ISP changes your home IP address on you! 
