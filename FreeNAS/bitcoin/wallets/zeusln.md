[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/extras.md) ]

## Zeus Lightning Wallet over Tor for Android

Tor has numerous benefits for serving your mobile wallet. 
* Securely, anonymously and remotely connect to your home server. 
* Connections are encrypted end to end.
* Onion address will remain static even if you are behind a dynamic IP address. No ddns required!
* No port forwards required by your router! 
* Tor v3 hidden services keep your `lnd` REST API a secret 
* Tor prevents denial of service attacks!

Download the Zeus app, APKs available here: https://github.com/ZeusLN/zeus/releases

View the private onion address of your tor hidden service [that you set up on this step!](https://github.com/seth586/guides/blob/master/FreeNAS/freenas_3_tor.md):
```
# cat /var/db/tor/remote_connections/hostname
myprivateonionaddressocyn4rixm632jid.onion
```

Download orbot for android. https://guardianproject.info/apps/orbot/

Open orbot, and select the gear icon under `tor enabled apps`. Add `Zeus`, then press back. Click `stop` on the big onion logo. Exit orbot and reopen it. Turn on `VPN Mode`. Start your connection to the tor network by clicking on the big onion (if it has not automatically connected already)


Link your lnd.conf to lnd's data dir:
```
# ln /usr/local/etc/lnd.conf /var/db/lnd/lnd.conf
```

Install `go`, compile [lndconnect](https://github.com/LN-Zap/lndconnect), and generate a LND connect URI QR code:
```
# pkg install go
# cd ~
# go get -d github.com/LN-Zap/lndconnect
# cd ~/go/src/github.com/LN-Zap/lndconnect
# gmake
# cd ~/go/bin
# ./lndconnect --lnddir=/var/db/lnd --image --host=z1234567890abc.onion --port=8080
```
A png file will be generated. Use WinSCP to download the image to your PC and scan it with the ZeusLN app!

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/extras.md) ]
