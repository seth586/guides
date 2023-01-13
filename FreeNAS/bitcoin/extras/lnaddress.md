[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor & i2p](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - **[Extras]** 



[Lightning Address](https://lightningaddress.com/), part of the LNURL spec [LUD16](https://github.com/lnurl/luds/blob/luds/16.md), allows you to receive payments to a static internet identifier ex: `you@domain.com`. While this is outside the LN spec & relies on traditional webserver infastructure (TLS, DNS, webserver, etc), [BOLT12](https://bolt12.org/) offers a potential native solution for static identifiers. We will use [ligess](https://github.com/Dolu89/ligess/) as our personal lightningaddress server.

## 1. Requirements:

A DNS resolved domain & TLS terminated webserver, such as the [reverse proxy](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) guide.

Lets get started!

## 2. Convert `invoice.macaroon` to hex in your `bitcoin` jail:
```
# hexdump -ve '1/1 "%.2x"' /var/db/lnd/data/chain/bitcoin/mainnet/invoice.macaroon
```

## 3. Create `lnaddress` jail

In this example I used static address `192.168.84.22` for my jail. SSH in: 
```
# pkg install nginx yarn-node18 git nano
# git clone https://github.com/dolu89/ligess
# cd ligess && yarn install
# cp .env.example .env
# nano .env
```
Edit .env: (adjust to your settings)
```
LIGESS_USERNAME=seth586
LIGESS_DOMAIN=example.com

LIGESS_LN_BACKEND=LND

LIGESS_LND_REST=https://192.168.84.21:8080
LIGESS_LND_MACAROON=INVOICEMACAROONHEXFORMATABC123ETC...
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

## 4. Set up nginx
```
# sysrc nginx_enable=yes
# nano /usr/local/etc/nginx/nginx.conf
```
Add the following location block to your server{} block and remove any other location {} blocks:
```
location /.well-known/lnurlp/seth586 {
        proxy_pass http://127.0.0.1:3000;
}
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

Start nginx ` service nginx start`

## 5. Test ligess
```
# cd /root/ligess && yarn dev
```
Open a web browser, and goto your jail `lnaddress` IP address:
```
http://192.168.84.22/.well-known/lnurlp/seth586
```
You should see a JSON response with the following details:
```
{"status":"OK","callback":"https://example.com/.well-known/lnurlp/seth586","tag":"payRequest","maxSendable":100000000,"minSendable":1000,"metadata":"[[\"text/identifier\",\"seth586@example.com\"],[\"text/plain\",\"Satoshis to seth586@example.com\"]]","commentAllowed":0}
```
Press Ctrl+C to terminate the process

## 6. rc.d script
`nano /usr/local/etc/rc.d/ligess`:
```
#!/bin/sh
#
# PROVIDE: ligess
# REQUIRE: 
# KEYWORD:

. /etc/rc.subr

name="ligess"
rcvar="ligess_enable"
ligess_chdir="/root/ligess"
ligess_command="/usr/local/bin/node /root/ligess/index.js"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${ligess_command}"

load_rc_config $name
: ${ligess_enable:=no}

run_rc_command "$1"
```
Save (CTRL+O, ENTER) and exit (CTRL+X)
```
# chmod +x /usr/local/etc/rc.d/ligess
# sysrc ligess_enable=yes
# service ligess start
```
Test again. Should work!

## 7. DNS resolve & TLS terminate
Lets make this reachable to the public internet and secure it with TLS to prevent man-in-the-middle attacks. 

SSH into your `reverseproxy` jail, create the following entry if you followed the [reverseproxy](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) guide

`nano /usr/local/etc/nginx/vdomains/example.com.conf`:

Add the following to your server{} block:
```
location /.well-known/lnurlp/seth586 {
        proxy_pass http://192.168.84.22:80;
}
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

Restart nginx `service nginx restart` and attempt over TLS connection:
```
https://example.com/.well-known/lnurlp/seth586
```

Try it out, send a tip to: `seth586@nym.im`

Note: if you want to receive, inbound liquidity on a private channel will not work. As of right now, ligess does [not support](https://github.com/Dolu89/ligess/issues/7) adding private routing hints.

[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]
