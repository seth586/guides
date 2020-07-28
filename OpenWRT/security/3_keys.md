[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - **[Generate Keys]** - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### Step 1a: Generate Keys without a FIDO/U2F device
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
User@Desktop ~/.ssh $ ls -la
drwx------+ 1 Seth None   0 Jul 27 23:42 .
drwxr-xr-x+ 1 Seth None   0 Jul  2 21:27 ..
-rw-------  1 Seth None 464 Jul 27 23:25 openwrt
-rw-r--r--  1 Seth None 108 Jul 27 23:25 openwrt.pub
User@Desktop ~/.ssh $ cat openwrt.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDLBdhdBaIlmBUAoVGT2PsGQyl5kTv1r+IJYIz1pVZsa DESKTOP-PCJ779K-27-07-2020
User@Desktop ~/.ssh $
```
`openwrt` is your private key and `openwrt.pub` is your public key. the `-a` flag requires 256 hash iterations to process your passphrase, this exponentially increases the processing power required to brute force your passphrase should your private key be compromised.

Repeat this step to create another keypair for your FreeNAS box, `User@Desktop ~/.ssh $ ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')" -f freenas`

### Step 1b: Generate Keys with a FIDO/U2F device
***work in progress***
Your OpenWRT router should be running the latest version of openssh, however FreeNAS runs an older version, so use Step 1a for your freenas keypair. But definately use this step for generating your OpenWRT keypair! On the next page, we will set the router up as a bastion host, requiring you to authenticate on the router before being able to connect to your freenas machine, effectively requiring FIDO/U2F device authentication to access freenas!

### Step 2: Add public keys to your OpenWRT Router
Highlight the result of `cat openwrt.pub` from Step 1a or 1b and copy it. We will paste it in the router's `~/.ssh/authorized-keys` file.
```
User@Desktop ~/.ssh $ ssh root@192.168.84.1 -p 2222
Password:
root@OpenWrt:~# nano ~/.ssh/authorized_keys
```
Paste your public key in a new line, save (CTRL+O, ENTER) and exit (CTRL+X). Now lets try to login using our new public private key authentication:
```
root@OpenWrt:~# exit
User@Desktop ~/.ssh $ ssh root@192.168.84.1 -p 22 -i openwrt
Enter passphrase for key '/home/User/.ssh/openwrt':
root@OpenWrt:~#
```
Success!

### Step 3: Add public key to your FreeNAS server:
Login to your freenas web-ui. Click "accounts", "users", "root", "edit". Paste the `cat freenas.pub` you copied to your clipboard from step 1a to the "SSH Public Key" field. Click "Save". Now attempt a SSH key based login, replace `192.168.84.85` with your freenas local IP address:
```
User@Desktop ~/.ssh $ ssh root@192.168.84.85 -p 22 -i freenas
Enter passphrase for key '/home/User/.ssh/freenas':
root@freenas:~# exit
User@Desktop ~/.ssh $
```
Success!

### Step 4: Create config file on client
Typing in `ssh root@192.168.84.1 -p 22 -i openwrt` is a lot of work, lets make things easier and set up a config file:
```
User@Desktop ~/.ssh $ touch config && chmod 600 config && nano config
```
Add the following info:
```
### The Bastion Host
Host openwrt
  HostName 192.168.84.1
  IdentityFile ~/.ssh/openwrt
  User root
  Port 22
### The Remote Host FreeNAS 
Host freenas
  HostName 192.168.84.85
  IdentityFile ~/.ssh/freenas
  User root
  Port 22
```
Save (CTRL+O, ENTER) and exit (CTRL+X). Test the config file:
```
User@Desktop ~/.ssh $ cd ~
User@Desktop ~ $ ssh openwrt
Enter passphrase for key '/home/User/.ssh/openwrt':
root@OpenWrt:~# exit
User@Desktop ~ $ ssh freenas
Enter passphrase for key '/home/User/.ssh/freenas':
root@freenas:~# exit
```
Success!

### Step 5: Disable and remove Dropbear on OpenWRT, turn off web-ui.
Remove dropbear:
```
root@OpenWrt:~# /etc/init.d/dropbear disable
root@OpenWrt:~# /etc/init.d/dropbear stop
root@OpenWrt:~# opkg remove dropbear
```
Stop the LUCI web interface, it is not required for your router to perform its functions. If you want to change router settings, then SSH in and start the web-ui, make your adjustments, then stop the web-ui.
```
/etc/init.d/uhttpd stop
```
Start the LUCI web interface:
```
/etc/init.d/uhttpd start
```

### Step 6: Disable SSH password authentication on FreeNAS
Login to your freenas web-ui. Click "accounts", "users", "root", "edit". Under "Disable Password", select "Yes". Click "Save". Now try a password based login:
```
User@Desktop ~ $ ssh root@192.168.84.85
root@192.168.84.85: Permission denied (publickey).
```
Success!

Next: [ [Bastion](4_bastion.md) ] >>
