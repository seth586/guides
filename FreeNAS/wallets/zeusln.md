[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

## Zeus Lightning Wallet over Tor for Android

Tor has numerous benefits for serving your mobile wallet. 
* Securely, anonymously and remotely connect to your home server. 
* Connections are encrypted end to end.
* Onion address will remain static even if you are behind a dynamic IP address. No ddns required!
* No port forwards required by your router! 
* Tor v3 hidden services keep your `lnd` REST API a secret 
* Tor prevents denial of service attacks!

Download the Zeus app, APKs available here: https://github.com/ZeusLN/zeus/releases, 
on Google Play and soon on F-Droid.

Log in to your RaspiBlitz through shh.

Edit `torrc` with `sudo nano /etc/tor/torrc` and add the following lines (`myandroid` can be unique):
```
HiddenServiceDir /mnt/hdd/tor/lnd_api/
HiddenServiceVersion 2
HiddenServiceAuthorizeClient stealth myandroid
HiddenServicePort 8080 127.0.0.1:8080
HiddenServicePort 10009 127.0.0.1:10009
```
Save (Ctrl+O, ENTER) and exit (Ctrl+X)

Restart Tor 
```
$ sudo systemctl restart tor
```

View the private credentials of your new hidden service. The first part is the onion address, the second part is the secret.
```
$ sudo cat /mnt/hdd/tor/lnd_api/hostname
z1234567890abc.onion AbyZXCfghtG+E0r84y/nR # client: myandroid
```

Download Orbot for Android. https://guardianproject.info/apps/orbot/

Open orbot. Click the `⋮`, select `hidden services ˃`, select `Client cookies`.

Press the + button on the lower right. Type in the the onion address and secret cookie you revealed with `sudo cat /mnt/hdd/tor/lnd_api/hostname`. Must enter onion address and add .onion to end in address area. For the cookie you
need all the information including [cookie] # client : [client] So for example:AbyZXCfghtG+E0r84y/nR # client: myandroid

Go back to orbot's main screen, and select the gear icon under `tor enabled apps`. Add `Zeus`, then press back. Click `stop` on the big onion logo. Exit orbot and reopen it. Turn on `VPN Mode`. Start your connection to the tor network by clicking on the big onion (if it has not automatically connected already)


Make sure go is installed (should be v1.11 or higher) :  
```
$ go version 
```
If need to install Go, run these:

```
$ wget https://storage.googleapis.com/golang/go1.11.linux-armv6l.tar.gz
$ sudo tar -C /usr/local -xzf go1.11.linux-armv6l.tar.gz
$ sudo rm *.gz
$ sudo mkdir /usr/local/gocode
$ sudo chmod 777 /usr/local/gocode
$ export GOROOT=/usr/local/go
$ export PATH=$PATH:$GOROOT/bin
$ export GOPATH=/usr/local/gocode
$ export PATH=$PATH:$GOPATH/bin
```

Download and compile [lndconnect](https://github.com/LN-Zap/lndconnect):
```
$ cd ~
$ go get -d github.com/LN-Zap/lndconnect
$ cd $GOPATH/src/github.com/LN-Zap/lndconnect
$ make
```
Generate the LND connect URI QR code:  
```
$ cd $GOPATH/bin
$ ./lndconnect --lnddir=/home/admin/.lnd --image --host=z1234567890abc.onion --port=8080
```
The file `lndconnect-qr.png` will be generated.   
  
In a Linux terminal run: [YOUR.RASIBLITZ.IP] = the local internal ip showing on your pi
``` 
$ scp admin@[YOUR.RASIBLITZ.IP]:$GOPATH/bin/lndconnect-qr.png ~/
```
and open the png from your home directory.  


May need to add something to view png from raspberry pi.
```
$ cd ~ 
$ sudo apt-get install fbi
$ sudo fbi -d /dev/fb0 -T 1 lndconnect-qr.png
```
Now plug in hdmi cord from pi to external monitor and you will see qr code to scan with zeus!

On Windows use WinSCP to download the image to your PC and open it.

Scan the QR Code with the ZeusLN app to be connected to your node through Tor!

SEND SATOSHI'S PRIVATE! Self Sovereignty for the streets!


[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
