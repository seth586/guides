[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

### Blockstream Green Wallet

This isn't a lightning app (yet), but you can connect it to your bitcoind node via SPV over tor!

Download and install Orbot for android: https://play.google.com/store/apps/details?id=org.torproject.android

Download the Blockstream Green app: https://play.google.com/store/apps/details?id=com.greenaddress.greenbits_android_wallet

Open orbot, and select the gear icon under "Tor-Enabled Apps". Select Green, then back. Turn on VPN Mode. Wait until the app is bootstrapped and connected to the tor network.

Open Green Wallet. Click on `Bitcoin ⯆`. Check `connect through a proxy`, with `Socks5 Hostname` `localhost` and `Socks5 Port` of `9050`. Check `Connect with Tor`. Click `Save`. 

Create a new wallet. Once you're done, Click the gear ⚙️ icon on the lower left. Under `Advanced` click `SPV synchronization`. Check `Enable SPV`. 
Select `Only connect to trusted node(s) for SPV`. Remember your bitcoind onion address? If not, SSH into your bitcoin jail, and run 
```
# bitcoin-cli -datadir=/var/db/bitcoin getnetworkinfo
```
Take that address, and put it in the trailing port number, `6aslkjdhfitshouldlooklikethis.onion:8333`
Finally, tap 'Reset SPV' and it will sync from your node. It may take a while to catch up. But this will privately connect to your home node, wherever you are!





[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
