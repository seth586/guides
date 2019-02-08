[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [**lnd**] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Install Lightning Lab's LND

#### 🚧🚧🚧THIS SECTION IS STILL UNDER CONSTRUCTION, DO NOT USE!🚧🚧🚧

If not already there, SSH into your freenas box as root, then switch to your bitcoin jail:
```
root@freenas[~] # iocage console bitcoin
```

Check [LND's github repo](https://github.com/lightningnetwork/lnd/releases) for the latest release, make sure you select the correct binaries for your processor and operating system. (amd64 is for amd and intel processors)
```
# cd ~
# wget https://github.com/lightningnetwork/lnd/releases/download/v0.5.2-beta/lnd-freebsd-amd64-v0.5.2-beta.tar.gz
# tar -xzf lnd-freebsd-amd64-v0.5.2-beta.tar.gz
# cd lnd-freebsd-amd64-v0.5.2-beta
# install -m 0755 -o root -g wheel lnd lncli /usr/local/bin
# cd ~
# rm -r lnd-freebsd-amd64-v0.5.2-beta
# mkdir /home/bitcoin/.lnd
# nano /home/bitcoin/.lnd/lnd.conf
```
### LND Configuration
Read up on configuration options [here](https://github.com/lightningnetwork/lnd/blob/master/sample-lnd.conf)
Add the following lines to your `lnd.conf` file:
```
[Application Options]
externalip=xxx.xxx.xxx.xxx
listen=0.0.0.0:9735
restlisten=0.0.0.0:8082
alias=thisisthenamethatblockexplorerswillshow
maxlogfiles=1
maxlogfilesize=10

[Bitcoin]
bitcoin.active=1
bitcoin.mainnet=1
bitcoin.node=bitcoind

[Bitcoind]
bitcoind.dir=/home/bitcoin/.bitcoin
```
Save (CTRL+O), then exit (CTRL+X)

Lets start it up!
```
# lnd --configfile=/home/bitcoin/.lnd/lnd.conf
```
If it works, you should see the following message:
```
root@btc:~ # lnd --configfile=/home/bitcoin/.lnd/lnd.conf
Attempting automatic RPC configuration to bitcoind
Automatically obtained bitcoind's RPC credentials
2019-02-07 22:00:34.994 [INF] LTND: Version: 0.5.2-beta commit=v0.5.2-beta, build=production, logging=default
2019-02-07 22:00:34.994 [INF] LTND: Active chain: Bitcoin (network=mainnet)
2019-02-07 22:00:35.013 [INF] CHDB: Checking for schema update: latest_version=7, db_version=7
2019-02-07 22:00:35.054 [INF] RPCS: password gRPC proxy started at [::]:8082
2019-02-07 22:00:35.054 [INF] RPCS: password RPC server listening on 127.0.0.1:10009
2019-02-07 22:00:35.054 [INF] LTND: Waiting for wallet encryption password. Use `lncli create` to create a wallet, `lncli unlock` to unlock an existing wallet, or `lncli changepassword` to change the password of an existing wallet and unlock it.
```
Press CTRL+C to exit.
