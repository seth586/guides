[ [<< Back to Extras](extras.md) ]

Download the Shango app, follow directions on their website: http://www.shangoapp.com/

Add the following line to your `/home/bitcoin/.lnd/lnd.conf` configuration file:
```
rpclisten=0.0.0.0:10009
```
Restart LND
```
# service lnd stop
# service lnd start
```
Remember to unlock your wallet, use RTL's web-ui.

You will need to convert your `/home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon` file into hex.
Unfortunately vim has a lot of dependencies, so you may want to do all of this in a seperate jail you can nuke later:

Use WinSCP to copy your `admin.macaroon' to a path in your new jail. Then run the following commands:

```
# pkg install vim
# xxd -p -c2000 ~/admin.macaroon
```

Use the hex output for your server credentials in the app.
[ [<< Back to Extras](extras.md) ]
