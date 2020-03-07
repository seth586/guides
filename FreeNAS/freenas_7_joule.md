[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

## Joule Browser Extension

### Install WinSCP
https://winscp.net/eng/download.php

With WinSCP, log into your freenas server using your root privileges, and navigate to the following folder (change as necessary):
```
/mnt/volume0/iocage/jails/bitcoin/root/var/db/lnd/data/chain/bitcoin/mainnet
```
Note: If you are unable to navigate to the `.lnd` folder, enable hidden folders by pressing (CTRL+ALT+H)

Download `admin.macaroon` and `readonly.macaroon`

### Install Joule

Goto Joule's website at https://lightningjoule.com/

Install the firefox extension on the [Tor Browser](https://www.torproject.org/download/). Click on the tor browser `3 line icon` on the top right, select `options`, and select privacy & `security`. Under `History` enable `remember hsitory`, otherwise the browser will not remember Joule's configuration.

Click on the joule extension button on the top right of your browser, and select `get started`. 

Select `remote node`

Type in your private onion address, port 8080:
```
https://myprivateonionaddressocyn4rixm632jid.onion:8080
```

You may have to follow the link since the browser does not trust lnd's TLS certificate by default. Once you follow the link, go back to joule's tab and it should work.

Upload `admin.macaroon` and `readonly.macaroon`, Make sure to delete these files, they give the bearer of these credentials the ability to spend your funds!

Set a password, and enjoy in-browser lighting payments, even away from your local home network! Try out a small tip here:
https://tippin.me/@seth586

If you want to try sending / receiving invoices, reach out to me. Contact info on the [ [Intro](README.md) ] page.

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
