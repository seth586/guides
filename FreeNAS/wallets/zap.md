[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

Download the Zap wallet on the Apple store: https://apps.apple.com/us/app/zap-bitcoin-lightning-wallet/id1406311960

Make sure the following line exists in your `lnd.conf` configuration file, if not, add and restart lnd (remmeber to unlock your wallet!):
```
rpclisten=127.0.0.1:8080
```

Link your lnd.conf to lnd's data dir:
```
ln /usr/local/etc/lnd.conf /var/db/lnd/lnd.conf
```

View the private onion address of your tor hidden service:
```
# cat /var/db/tor/remote_connections/hostname
whateveryouronionaddressis.onion
```

Install `go`, compile [lndconnect](https://github.com/LN-Zap/lndconnect), change `--host=` to the jails local IP, and generate a LND connect URL QR code:
```
# pkg install go
# cd ~
# go get -d github.com/LN-Zap/lndconnect
# cd ~/go/src/github.com/LN-Zap/lndconnect
# gmake
# cd ~/go/bin
# ./lndconnect --lnddir=/var/db/lnd --image --host=whateveryouronionaddressis.onion --port=8080
```
A png file will be generated. Use WinSCP to download the image (path should be something like this: `/mnt/volume0/iocage/jails/bitcoin/root/root/go/bin`)to your PC and scan it with the zap app!

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
