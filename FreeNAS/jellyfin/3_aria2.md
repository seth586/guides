[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [Jellyfin](2_jellyfin.md) ] - **[aria2]** - [ [Medusa](4_medusa.md) ]

```
pkg install nano aria2 lighttpd ca_root_nss git
sysrc lighttpd_enable=YES
sysrc aria2_enable=YES
sysrc aria2_user=jellyfin
sysrc aria2_group=jellyfin
mkdir /var/db/aria2
touch /var/db/aira2/aria2.txt
chown -R jellyfin:jellyfin /var/db/aria2
chmod -R 770 /var/db/aria2
nano /usr/local/etc/aria2.conf
```
Configure: (See configuration options [here](https://aria2.github.io/manual/en/html/aria2c.html#aria2-conf))
```
continue=true
daemon=true
dir=/media/downloads
file-allocation=none
log-level=warn
disable-ipv6=true
log-level=warn
max-connection-per-server=4
max-concurrent-downloads=3
max-overall-download-limit=0
min-split-size=5M
rpc-listen-all=true
enable-rpc=true
rpc-secret=t2150cdt!
ca-certificate=/etc/ssl/cert.pem
save-session=/var/db/aria2/aria2.txt
input-file=/var/db/aria2/aria2.txt
save-session-interval=10
dht-file-path=/var/db/aria2/dht.dat
dht-file-path6=/var/db/aria2/dht6.dat
max-overall-upload-limit=125K
seed-ratio=1.0
seed-time=1440
```
Save (Ctrl+O, ENTER) and exit (Ctrl + X)
```
service aria2 start
```
Point your Aria2 front end to  http://yourJailIP:6800/jsonrpc
