[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/extras.md) ]

## Zeus Lightning Wallet over Tor for Android

ZeusLN has tor built in, orbot isnt required!

Download the Zeus app, APKs available here: https://github.com/ZeusLN/zeus/releases

View the private onion address of your tor hidden service [that you set up on this step!](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/freenas_3_tor.md):
```
# cat /var/db/tor/remote_connections/hostname
myprivateonionaddressocyn4rixm632jid.onion
```

If you followed the guide, your port should be `8080`

### Create lndconnect qr in-terminal
Replace the IP address with your `*.onion` address, or use local IP if you have a VPN to your home network:
```
echo 'lndconnect://192.168.84.21:8080?macaroon='"`base64 /var/db/lnd/data/chain/bitcoin/mainnet/admin.macaroon | tr -d '=' | tr '/+' '_-'`" | tr -d '\n' | qrencode -m 2 -t utf8
```
Or dump the admin.macaroon hex and copy into your app:
```
hexdump -ve '1/1 "%.2x"' /var/db/lnd/data/chain/bitcoin/mainnet/admin.macaroon
```

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/extras.md) ]
