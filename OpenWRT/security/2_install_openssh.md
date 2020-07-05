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

### Install & Configure OpenSSH Server on OpenWRT
Lets search to see what versions are avialble with our default package repository:
```
root@OpenWrt:~# opkg list | grep openssh
```
What versions do you see? As of writing, Version 8.0. Not good enough! We need at minimum version 8.1 to support interactive ed25519-sk signature types! Fortunately someone has been compiling newer software versions for the Linksys WRT line, check out [Davidc502 OpenWrt snapshots](https://dc502wrt.org/). Specifically, the website owner has a repository at https://dc502wrt.org/snapshots/. Click on the latest release (r13342 as of writing)->(packages)->(arm_cortex-a9_vfpv3-d16)->(Packages). Scroll down to the openssh packages. Looks like version 8.2 compiled for our hardware, excellent! Copy the URL and paste using the `curl -O` command and install:
```
root@OpenWrt:~# curl -O https://dc502wrt.org/snapshots/r13342/packages/arm_cortex-a9_vfpv3-d16/packages/openssh
-server_8.2p1-3_arm_cortex-a9_vfpv3-d16.ipk
root@OpenWrt:~# opkg install openssh-server_8.2p1-3_arm_cortex-a9_vfpv3-d16.ipk
root@OpenWrt:~# rm openssh-server_8.2p1-3_arm_cortex-a9_vfpv3-d16.ipk
root@OpenWrt:~# nano /etc/ssh/sshd_config
```
Change the following lines:
```
PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication no
```
Save (CTRL+O, ENTER) and Exit (CTRL+X). Now enable and start the OpenSSH server
```
root@OpenWrt:~# /etc/init.d/sshd enable
root@OpenWrt:~# /etc/init.d/sshd start
root@OpenWrt:~# sshd -v
OpenSSH_8.2p1, OpenSSL 1.1.1g  21 Apr 2020
```
Success!

Now lets create the `~/.ssh` folder which will hold our public keys, set [permissions](https://www.freebsd.org/doc/handbook/permissions.html), and exit to our client:
```
root@OpenWrt:~# mkdir ~/.ssh
root@OpenWrt:~# touch ~/.ssh/authorized_keys
root@OpenWrt:~# chmod 700 ~/.ssh
root@OpenWrt:~# chmod 600 ~/.ssh/*
```
### Install OpenSSH Client on OpenWRT
Since it is our goal to use our router as a SSH bastion, we will install the OpenSSH client on OpenWRT so that we must authenticate on our OpenWRT bastion before making further connections to our servers such as FreeNAS / TrueNAS. 
```
root@OpenWrt:~# curl -O https://dc502wrt.org/snapshots/r13342/packages/arm_cortex-a9_vfpv3-d16/packages/openssh-client_8.2p1-3_arm_cortex-a9_vfpv3-d16.ipk
root@OpenWrt:~# opkg install openssh-client_8.2p1-3_arm_cortex-a9_vfpv3-d16.ipk
root@OpenWrt:~# rm openssh-client_8.2p1-3_arm_cortex-a9_vfpv3-d16.ipk
```
Verify our ssh client works by attempting a password authentication to our FreeNAS server (replace `192.168.84.85` with your freenas server local IP address):
```
root@OpenWrt:~# ssh root@192.168.84.85
```

Next: [ [Generate Keys](3_keys.md) ] >>
