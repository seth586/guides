[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [Jellyfin](2_jellyfin.md) ] - [ [aria2](3_aria2.md) ] - **[Medusa]**

```
pkg update && pkg upgrade
pkg install python39 py39-sqlite3 unrar git openssl
pw adduser medusa -G media -d /nonexistent -s /usr/sbin/nologin
git clone https://github.com/pymedusa/Medusa.git /usr/local/medusa
cp /usr/local/medusa/runscripts/init.freebsd /usr/local/etc/rc.d/medusa
chmod 755 /usr/local/etc/rc.d/medusa
chown -Rf medusa:medusa /usr/local/medusa
sysrc "medusa_enable=YES"
service medusa start
```
