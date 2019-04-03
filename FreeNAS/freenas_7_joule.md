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

### Optional: Remote connections over Tor
Want to use Joule on your laptop, while away from your home network? Tor can give you anonymous remote connectivity!

Download the Tor Browser [here](https://www.torproject.org/download/). The Tor browser is based on Firefox, so Joule's extension also works on the Tor browser!

In your bitcoin jail, edit `torrc` with `nano /usr/local/etc/tor/torrc` and add the following lines (`mydevice` can be unique):

```
HiddenServiceDir /var/db/tor/lnd_api/
HiddenServiceVersion 2
HiddenServiceAuthorizeClient stealth mydevices
HiddenServicePort 8082 127.0.0.1:8082
HiddenServicePort 10009 127.0.0.1:10009
``` 
Save (Ctrl+O, ENTER) and exit (Ctrl+X)

Restart Tor 
```
# service tor restart
```

View the private credentials of your new hidden service. The first part is the onion address, the second part is the password(cookie).
```
# cat /var/db/tor/lnd_api/hostname
z1234567890abc.onion AbyZXCfghtG+E0r84y/nR # client: mydevices
```

On your windows laptop, the tor browser will create a folder on your desktop. Navigate to `\Desktop\Tor Browser\Browser\TorBrowser\Data\Tor\` and edit the `torrc` file. 
Add the following line. Replace the onion address, password(cookie), and mydevice with your credentials:
```
HidServAuth z1234567890abc.onion AbyZXCfghtG+E0r84y/nR mydevices
```

Close and reopen the tor browser. Install the [Joule extension for firefox](https://lightningjoule.com/). Lets turn on our browsing history so the browser can remember that we approved our self signed TLS security certificate. Click the 3-line button on the top right, click `options`, and click `privacy & security`.  Under History, select Tor Browser will 'Remember history'.

Now go ahead and set up Joule as described above, except use the tor address revealed by `cat /var/db/tor/lnd_api/hostname'. Example:
```
https://z1234567890abc.onion:8082
```

Next: [ [Extras](extras.md) ]
