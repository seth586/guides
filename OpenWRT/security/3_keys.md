[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - **[Generate Keys]** - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### Generate Keys without a FIDO/U2F device
Log in to your client device and generate a key with [ssh-keygen](https://man.openbsd.org/OpenBSD-current/man1/ssh-keygen.1#NAME):
```

User@Desktop ~ $ cd ~/.ssh
User@Desktop ~/.ssh $ ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')" -f openwrt
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in openwrt
Your public key has been saved in openwrt.pub
The key fingerprint is:
SHA256:rrMD+qPUmwwuSgXqP2vKuJAzzz1NI+Vypen0+RCbeqA DESKTOP-PCJ779K-27-07-2020
The key's randomart image is:
+--[ED25519 256]--+
|                 |
|                 |
| .               |
|. .   . .        |
|.  . o +S        |
|....+ X. +       |
|=oo..@ +=.       |
|*Bo*Eo=o+.       |
|==B=Oo+= ..      |
+----[SHA256]-----+
User@Desktop ~/.ssh $ ls
openwrt openwrt.pub
User@Desktop ~/.ssh $ cat openwrt.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDLBdhdBaIlmBUAoVGT2PsGQyl5kTv1r+IJYIz1pVZsa DESKTOP-PCJ779K-27-07-2020
User@Desktop ~/.ssh $
```
`openwrt` is your private key and `openwrt.pub` is your public key. the `-a` flag requires 256 hash iterations to process your passphrase, this exponentially increases the processing power required to brute force your passphrase should your private key be compromised.

Highlight the result of `cat openwrt.pub` and copy it. We will paste it in:

```
User@Desktop ~/.ssh $ ssh root@192.168.84.1 -p 2222
Password:
root@OpenWRT
```

### Generate Keys with a FIDO/U2F device

### Attempt Connections Individually

### Disable and remove Dropbear on OpenWRT
```
root@OpenWrt:~# /etc/init.d/dropbear disable
root@OpenWrt:~# /etc/init.d/dropbear stop
root@OpenWrt:~# opkg remove dropbear
```

Next: [ [Bastion](4_bastion.md) ] >>
