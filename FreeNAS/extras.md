[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ] -  **[Extras]** 

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### My Extras

#### [Configure OpenWRT with UPNP for 'lnd' auto update external ip address](https://github.com/seth586/guides/blob/master/OpenWRT/upnp_natpmp.md)
Don't have a static IP address? `lnd` will loose peer connections unless you configure `nat=true` and have a working UPnP implementation. This is a process to ensure your UPnP implementation is secure.

#### [Shango - Android / iOS mobile wallet](wallets/shango.md)

### External guides
Want to use a lightning wallet, like [Shango](http://www.shangoapp.com/) away from your home network? Here are a few options on setting up a VPN server at home, so you can securely connect to your `lnd` on the road!

#### [SoftEther VPN in a FreeNAS jail](https://forums.freenas.org/index.php?threads/alternative-to-openvpn-softether-vpn.47395/)
SoftEther offers a free DNS service, great option if your ISP changes your home IP address on you! 

#### [Tor relay node](https://trac.torproject.org/projects/tor/wiki/TorRelayGuide/FreeBSD) 
Tor doesn't give anonymity unless there are sufficient relays to hide the source from the destination, and vice versa. Do NOT run a Tor exit, run a relay only. Make sure `ExitRelay 0` is set! I also recommend `RelayBandwidthRate 250 KBytes` and `MaxAdvertisedBandwidth 250 Kbytes` so it doesn't eat up all of your bandwidth. (125kbyte = 1 Mbit, adjust as you see fit). Run this in its own jail. Port forward 9001 to your tor relay jail. If you need an onion service anonymity (such as running bitcoin in `onlynet=onion`) [do not run a relay](https://research.kudelskisecurity.com/2013/09/04/dont-run-a-tor-router-and-a-hidden-service-from-the-same-connection/).
