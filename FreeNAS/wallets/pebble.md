[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

Download the pebble wallet for your mobile device: https://www.pebble.indiesquare.me/

Link your lnd.conf to lnd's data dir:
```
# ln /usr/local/etc/lnd.conf /var/db/lnd/lnd.conf
```

Install `go`, compile [lndconnect](https://github.com/LN-Zap/lndconnect), and generate a LND connect URL QR code:
```
# pkg install go
# cd ~
# go get -d github.com/LN-Zap/lndconnect
# cd ~/go/src/github.com/LN-Zap/lndconnect
# gmake
# cd ~/go/bin
# ./lndconnect --lnddir=/var/db/lnd --image --host=192.168.84.123
```
A png file will be generated. Use WinSCP to download the image to your PC and scan it with the pebble app!

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
