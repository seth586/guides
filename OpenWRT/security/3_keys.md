[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - **[Generate Keys]** - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### Generate Keys

### Attempt Connections Individually

### Disable and remove Dropbear on OpenWRT
```
root@OpenWrt:~# /etc/init.d/dropbear disable
root@OpenWrt:~# /etc/init.d/dropbear stop
root@OpenWrt:~# opkg remove dropbear
```

Next: [ [Bastion](4_bastion.md) ] >>
