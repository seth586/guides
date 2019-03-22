[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

Tor has numerous benefits for serving your mobile wallet. 
* Securely, anonymously and remotely connect to your home server. 
* Connections are encrypted
* Onion address will remain static even if you are behind a dynamic IP address. No ddns required!
* No port forwards required by your router! 
* Tor cookie authenticated hidden services keep your `lnd` REST API a secret
* Tor cookie authentication prevents denial of service attacks

Download the Zeus app, APKs available here: https://github.com/ZeusLN/zeus/releases

Edit `torrc` with `nano /usr/local/etc/tor/torrc` and add the following lines (`myandroid` can be unique):
```
HiddenServiceDir /var/db/tor/lnd_rest/
HiddenServiceVersion 2
HiddenServiceAuthorizeClient stealth myandroid
HiddenServicePort 8082 127.0.0.1:8082
```
Save (Ctrl+O, ENTER) and exit (Ctrl+X)

Restart Tor 
```
# service tor restart
```

View the private credentials of your new hidden service. The first part is the onion address, the second part is the secret.
```
# cat /var/lnd/tor/lnd_rest/hostname
z1234567890abc.onion AbyZXCfghtG+E0r84y/nR # client: myandroid
```

Download orbot for android. https://guardianproject.info/apps/orbot/

Open orbot. Click the `⋮`, select `hidden services ˃`, select `Client cookies`.

Press the + button on the lower right. Type in the the onion address and secret cookie you revealed with `cat  /var/lnd/tor/lnd_rest/hostname`.

Go back to orbot's main screen, and select the gear icon under `tor enabled apps`. Add `Zeus`, then press back. Click `stop` on the big onion logo. Exit orbot and reopen it. Turn on `VPN Mode`.


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
# ./lndconnect --lnddir=/var/db/lnd --image --host=z1234567890abc.onion
```
A png file will be generated. Use WinSCP to download the image to your PC and scan it with the ZeusLN app!

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
