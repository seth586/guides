[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [**Joule**] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

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

Install the chrome extension.

Click on the joule extension button on the top right of your browser, and select `get started`. 

Select `remote node`

Type in your bitcoin jail address, port 8082 as described in our `'lnd.conf' file's REST configuration.
```
https://192.168.84.123:8082
```

You may have to follow the link since the browser does not trust lnd's TLS certificate by default. Once you follow the link, go back to joule's tab and it should work.

Upload `admin.macaroon` and `readonly.macaroon`, Make sure to delete these files, they give the bearer of these credentials the ability to spend your funds!

Set a password, and enjoy in-browser lighting payments! Try out a small tip here:
https://tippin.me/@seth586

If you want to try sending / receiving invoices, reach out to me. Contact info on the [ [Intro](README.md) ] page.
