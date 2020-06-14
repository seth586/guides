[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/extras.md) ]

### BlueWallet

BlueWallet for android serves layer 1 on-chain transactions over the electrum protocol, which is more data efficient compared to SPV (such as Blockstream Green). We can serve this function with our own electrum server! The BlueWallet roadmap plans future support for BIP174, which would allow a hardware wallet like the coldcard to be utilized on android!

BlueWallet serves layer 2 lightning transactions with a wrapper called [lndhub](https://github.com/BlueWallet/LndHub) which is designed to share an LND node in a trusted environment. Read up on the technical tradeoffs bluewallet employs with lndhub [here](https://medium.com/bluewallet/bluewallet-brings-zero-configuration-lightning-payments-to-ios-and-android-30137a69f071). This guide does not explain how to set up your own lndhub.

Download and install Orbot for android: https://play.google.com/store/apps/details?id=org.torproject.android

Download the BlueWallet app: https://bluewallet.io/

Open orbot, and select the gear icon under "Tor-Enabled Apps". Select BlueWallet, then back. Turn on VPN Mode. Wait until the app is bootstrapped and connected to the tor network.

Remember your electrum onion address? If not, SSH into your bitcoin jail, and run 
```
# cat /var/db/tor/remote_connections/hostname
```

Open BlueWallet on your android device. Click on the hamburger menu for settings `☰`. Click `Electrum server >`, set the address to your onion address and port to '50002'. 

You should get a sucessful connection! Now import your electrum & hardware wallet ypub for a watch / receive only wallet on android verified by your own electrum server, or use it as a full wallet!





[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/extras.md) ]
