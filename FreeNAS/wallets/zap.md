[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

Download the Zap wallet on the Apple store: https://testflight.apple.com/join/P32C380R

Add the following line to your `lnd.conf` configuration file:
```
rpclisten=0.0.0.0:10009
```
Restart LND
```
# service lnd stop
# service lnd start
```
Remember to unlock your wallet, use RTL's web-ui.

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
A png file will be generated. Use WinSCP to download the image to your PC and scan it with the zap app!

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
