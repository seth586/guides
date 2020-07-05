[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - **[OpenSSH on OpenWRT]** - [ [Generate Keys](3_keys.md) ] - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### Install OpenSSH on OpenWRT
By default OpenWRT comes installed with a lite SSH server called `dropbear`. However if we want to use more advanced features like FIDO/U2F, we will need the heavier & full featured `OpenSSH`. Don't worry, if you're running a modern OpenWRT enabled router you'll be just fine!

### Reconfigure dropbear
Lets start with changing the default listening port of `dropbear` from port 22 to 2222. That way we can make sure our OpenWRT public key authentication works before we remove our password based authentication.

SSH into your router.
```
$ SSH root@192.168.84.1
root@OpenWrt:~# opkg update
...
root@OpenWrt:~# opkg install nano
...
root@OpenWrt:~# nano /etc/config/dropbear
```
Change `option port '22'` to `option port '2222'`. Save (CTRL+O,ENTER) and Exit (CTRL+X). Restart dropbear and exit ssh session:
```
root@OpenWrt:~# /etc/init.d/dropbear restart
root@OpenWrt:~# exit
Connection to 192.168.84.1 closed.
User@DESKTOP ~
$
```
Relogin using the new port to verify dropbear still works:
```
$ ssh root@192.168.84.1 -p 2222
```

### Install OpenSSH
