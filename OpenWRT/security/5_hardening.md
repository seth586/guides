[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - [ [Generate Keys](3_keys.md) ] - [ [Bastion](4_bastion.md) ] - **[Hardening]**

## SSH Bastion & Security Hardening Guide
### Upgrade opk to https connections
```
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install wget
root@OpenWrt:~# opkg install ca-certificates
root@OpenWrt:~# opkg install libustream-openssl
root@OpenWrt:~# nano /etc/opkg/distfeeds.conf
```
Replace all `http` urls with `https`. Save (CTRL+O, ENTER) and exit (CTRL+X)
```
root@OpenWrt:~# opkg update
```
You should see update lsits download from https addresses and see `signature check passed`.

[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]
