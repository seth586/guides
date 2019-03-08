[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

Download the Zeus app, APKs available here: https://github.com/ZeusLN/zeus/releases

You will need to convert your `/var/db/lnd/data/chain/bitcoin/mainnet/admin.macaroon` file into hex.
Unfortunately `vim` has a lot of dependencies, so you may want to do all of this in a seperate jail you can nuke later:

Use WinSCP to copy your `admin.macaroon' to a path in your new jail. Then run the following commands:

```
# pkg install vim
# xxd -ps -u -c 1000 /path/to/admin.macaroon
```

Use the hex output for your server credentials in the app. Use REST API port `8082` and your local ip address for your bitcoin jail.

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
