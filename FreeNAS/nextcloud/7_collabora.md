[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [mariadb](2_mariadb.md) ] - [ [PHP](3_php.md) ] - [ [apache](4_apache.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ **collabora** ]

## Guide to Nextcloud server on TrueNAS

https://github.com/CollaboraOnline/online/blob/master/.cirrus.yml

https://collaboraonline.github.io/post/build-code/ Has instructions how to build CODE


### 1. Set up environment

Create a new jail `collabora`, give it a static IP. SSH into the jail

`pkg install -y nano && mkdir -p /usr/local/etc/pkg/repos/ && nano /usr/local/etc/pkg/repos/FreeBSD.conf`:
```
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```

### 2. install prerequisites & build LibreOffice 
```
# pkg install -y python3 python38 py38-polib py38-lxml gmake pkgconf poco cppunit autotools coreutils git bash npm png pango
# # git clone --single-branch --branch distro/collabora/co-2021 https://gerrit.libreoffice.org/core /root/libreoffice
# cd /root/libreoffice

# cd /usr/ports/editors/libreoffice/work/libreoffice-7.3.2.2
# fetch https://github.com/CollaboraOnline/online/releases/download/for-code-assets/LibreOfficeKit-includes-co-2021.tar.gz
# tar -xzf LibreOfficeKit-includes-co-2021.tar.gz
# rm LibreOfficeKit-includes-co-2021.tar.gz
```

Note directory `/usr/ports/editors/libreoffice/work/libreoffice-7.3.2.2/include` 
and `/usr/ports/editors/libreoffice/work/libreoffice-7.3.2.2/instdir `

### 3. build Collabora Online 

```
# cd ~
# git clone https://github.com/CollaboraOnline/online collabora-online
# cd collabora-online
# pw useradd -n cool -d /tmp/coolhome -m
# chmod -R o+rwx ./
# su -m cool -c './autogen.sh'
# su -m cool -c 'env HOME=/tmp/coolhome MAKE=gmake CPPFLAGS="-isystem /usr/local/include" CFLAGS="-I/usr/local/include" CXXFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" ./configure --with-lo-path=/usr/local/lib/libreoffice/ --with-lokit-path=/usr/ports/editors/libreoffice/work/libreoffice-7.3.2.2/include --disable-seccomp --disable-setcap --enable-debug'
# su -m cool -c 'env HOME=/tmp/coolhome gmake -j`sysctl -n hw.ncpu`'
# chown root ./coolmount
# chmod +s ./coolmount
# su -m cool -c 'env HOME=/tmp/coolhome gmake check'
```

### 4. run 
```
# chown cool /tmp/coolwsd.log
# su -m cool -c 'gmake run'
```
