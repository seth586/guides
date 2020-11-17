[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [**Electrum**] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & Lightning️ on FreeNAS / TrueNAS ![BSDBTC60.png](images/BSDBTC60.png)

### Electrs: Electrum In Rust

Read up more on electrs at its github page [here](https://github.com/romanz/electrs)


TrueNAS 12's base compiler is llvm10, however we need to downgrade to version 9 for a sucessful compile. Download prerequisites and compile:
```
# pkg install rust git llvm90 nano
# ln /usr/local/llvm90/bin/clang++ /usr/local/llvm90/bin/c++
# nano ~/.cshrc
```
Add the `/usr/local/llvm90/bin` directory to a priority position in the shell's path:
```
set path = (/usr/local/llvm90/bin /sbin /bin /usr/sbin /usr/bin /usr/local/sbin /usr/local/bin $HOME/bin)
```
Save (CTRL+O, ENTER) and exit (CTRL+X). Exit the jail by typing `exit`, then re-enter the jail `iocage console bitcoin` so the shell loads the new configuration.

Now lets download and compile!
```
# cd ~
# git clone https://github.com/romanz/electrs
# cd electrs
# cargo build --release
```

If you get an error, see this [issue for FreeBSD systems](https://github.com/romanz/electrs/issues/132#issuecomment-481870879) to fix. Rerun `cargo build --release`

Install and cleanup:
```
# install -m 0755 -o root -g wheel /root/electrs/target/release/electrs /usr/local/bin
# rm -r ~/electrs
# mkdir /var/db/electrs
# electrs -vvv --db-dir=/var/db/electrs --electrum-rpc-addr=192.168.84.208:50001 --daemon-dir=/var/db/bitcoin
```
Make sure to replace `electrum-rpc-addr=` with your bitcoin jail's IP to serve local connections, or use 127.0.0.1:50001 if you plan on serving remote connections over tor authenticated hidden services.

Electrs will now index the blockchain into its own database. This can take a few hours, depending on your CPU and disk IO. You may get spammed by a `WARN - failed to export stats: failed to read stats`, just ignore it. When its done indexing, it will start to serve connections.

Right click on your windows electrum client, select properties, and modify the shortcut to resemble below:
```
"C:\Program Files (x86)\Electrum\electrum-3.3.4.exe" -1 -s 192.168.84.208:50001:t
```
Start up electrum client. It should connect! Terminate electrs with ctrl+c. Verify it is no longer running with `ps aux`. If it is still running, kill it with `kill -9 <pid>`, whereas `<pid>` is the number under the PID column from command `ps aux`. 

Now lets write an rc.d script so it automatically starts as a service:

```
# nano /usr/local/etc/rc.d/electrs
```

Paste the following script:
```
#!/bin/sh
#
# PROVIDE: electrs
# REQUIRE: bitcoind
# KEYWORD:

. /etc/rc.subr

name="electrs"
rcvar="electrs_enable"
electrs_command="/usr/local/bin/electrs --db-dir=/var/db/electrs --electrum-rpc-addr=192.168.84.208:50001 --daemon-dir=/var/db/bitcoin"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${electrs_command}"

load_rc_config $name
: ${electrs_enable:=no}

run_rc_command "$1"
```
Save, (ctrl+o,enter) and exit (ctrl+x)

Make the script executable:
```
# chmod +x /usr/local/etc/rc.d/electrs
```
And enable on startup:
```
# sysrc electrs_enable="YES"
```
Give it a whir:
```
# service electrs start
```

### How to update electrs
```
# service electrs stop
# pkg update
# pkg upgrade rust
# cd ~
# git clone https://github.com/romanz/electrs
# cd electrs
# cargo build --release
# install -m 0755 -o root -g wheel /root/electrs/target/release/electrs /usr/local/bin
# rm -r ~/electrs
# service electrs start
```

### Bonus: Reverse proxy configuration for domain & SSL certificate access
If you want to access electrum without using a VPN or TOR, you can have a SSL encrypted connection by configuring your [reverse proxy](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md) config file by appending the data below to your main nginx config file located at `/usr/local/etc/nginx/nginx.conf` in your reverseproxy jail:
```
### ELECTRUM.EXAMPLE.COM
stream {
        upstream electrs {
                server 192.168.84.208:50001;
        }

        server {
                listen 50002 ssl;
                proxy_pass electrs;
                ssl_certificate /usr/local/etc/letsencrypt/live/example.com/fullchain.pem;
                ssl_certificate_key /usr/local/etc/letsencrypt/live/example.com/privkey.pem;
                ssl_session_cache shared:SSL:1m;
                ssl_session_timeout 4h;
                ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
                ssl_prefer_server_ciphers on;
        }
}
### / ELECTRUM
```
Change `192.168.84.208` with the jail hosting electrum. Save (CTRL+O, ENTER) and exit (CTRL+X), and refresh nginx with `service nginx restart`

Next: [ [lnd](freenas_5_lnd.md) ]
