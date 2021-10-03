[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [Jellyfin](2_jellyfin.md) ] - [ [aria2](3_aria2.md) ] - **[Medusa]**


```
cat /etc/passwd
pw groupshow -a
pw adduser medusa -G jellyfin -d /nonexistent -s /usr/sbin/nologin

```


```
pkg update && pkg upgrade
pkg install python39 py39-sqlite3 unrar git openssl


git clone https://github.com/pymedusa/Medusa.git /usr/local/medusa
cp /usr/local/medusa/runscripts/init.freebsd /usr/local/etc/rc.d/medusa
chmod 755 /usr/local/etc/rc.d/medusa
chown -Rf medusa:medusa /usr/local/medusa
sysrc "medusa_enable=YES"
service medusa start
```
