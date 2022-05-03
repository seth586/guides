[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [mariadb](2_mariadb.md) ] - [ [PHP](3_php.md) ] - [ [apache](4_apache.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ **collabora** ]

## Guide to Nextcloud server on TrueNAS

https://collaboraonline.github.io/post/build-code/ Has instructions how to build CODE

### 1. install prerequisites
```
# portsnap fetch && portsnap extract
# cd /usr/ports/editors/libreoffice && make patch
# make -DBATCH install clean
```

Create a new jail `collabora`, give it a static IP. SSH into the jail

`pkg install -y nano && mkdir -p /usr/local/etc/pkg/repos/ && nano /usr/local/etc/pkg/repos/FreeBSD.conf`:
```
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```

```
# pkg install -y python3 python38 py38-polib py38-lxml
# pkg install -y gmake pkgconf poco cppunit autotools coreutils git bash npm png pango



# mkdir libreoffice-src
# cd libreoffice-src
# fetch https://github.com/CollaboraOnline/online/releases/download/for-code-assets/LibreOfficeKit-includes-co-2021.tar.gz
# tar -xzf LibreOfficeKit-includes-co-2021.tar.gz
# mkdir -p .git/hooks
# pw useradd -n cool -d /tmp/coolhome -m
# chmod -R o+rwx ./
# su -m cool -c './autogen.sh'
# su -m cool -c ''env HOME=/tmp/coolhome MAKE=gmake
        CPPFLAGS="-isystem /usr/local/include" CFLAGS="-I/usr/local/include"
        CXXFLAGS="-I/usr/local/include" LDFLAGS=-L/usr/local/lib ./configure
        --with-lo-path=/usr/local/lib/libreoffice/
        --with-lokit-path=./libreoffice-src/include
        --disable-seccomp --disable-setcap --enable-debug''
# su -m cool -c 'env HOME=/tmp/coolhome gmake -j`sysctl -n hw.ncpu`
# chown root ./coolmount
# chmod +s ./coolmount
# su -m cool -c 'env HOME=/tmp/coolhome gmake check'
```
