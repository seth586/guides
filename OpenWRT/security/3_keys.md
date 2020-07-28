[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - **[Generate Keys]** - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### Step 1a: Generate Keys without a FIDO2 device
Log in to your client device and generate a key with [ssh-keygen](https://man.openbsd.org/OpenBSD-current/man1/ssh-keygen.1#NAME):
```
User@Desktop ~ $ ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')" -f ~/.ssh/openwrt
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
User@Desktop ~ $ cat ~/.ssh/openwrt.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDLBdhdBaIlmBUAoVGT2PsGQyl5kTv1r+IJYIz1pVZsa DESKTOP-PCJ779K-27-07-2020
User@Desktop ~ $
```
`openwrt` is your private key and `openwrt.pub` is your public key. the `-a` flag requires 256 hash iterations to process your passphrase, this exponentially increases the processing power required to brute force your passphrase should your private key be compromised.

Repeat this step to create another keypair for your FreeNAS box, 
```
User@Desktop ~ $ ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')" -f ~/.ssh/freenas
``` 
You should now have the following files in `~/.ssh`:
```
User@Desktop ~ $ ls -la ~/.ssh
drwx------+ 1 Seth None   0 Jul 27 23:42 .
drwxr-xr-x+ 1 Seth None   0 Jul  2 21:27 ..
-rw-------  1 Seth None 464 Jul 27 23:25 openwrt
-rw-r--r--  1 Seth None 108 Jul 27 23:25 openwrt.pub
-rw-------  1 Seth None 464 Jul 27 23:25 freenas
-rw-r--r--  1 Seth None 108 Jul 27 23:25 freenas.pub
```

### Step 1b: Generate Keys with a FIDO2 device
Your OpenWRT router should be running the latest version of openssh, however FreeNAS runs an older version, so use Step 1a for your freenas keypair. But definately use this step for generating your OpenWRT keypair! On the next page, we will set the router up as a bastion host, requiring you to authenticate on the router before being able to connect to your freenas machine, effectively requiring FIDO2 device authentication to access freenas!

It is highly recommended you have two or more FIDO2 devices, you dont want a single point of failure! Run this command for each FIDO2 device, and set the `-f` filename uniquely per device:
```
User@Desktop ~ $ ssh-keygen -t ed25519-sk -C "$(hostname)-$(date +'%d-%m-%Y')-yubikey_description" -f ~/.ssh/openwrt_yubi
```

All other steps are the same as 1a.

### Step 2: Add public keys to your OpenWRT Router
Highlight the output of your public key from the command `cat openwrt.pub` from Step 1a to copy it. We will paste it in the router's `~/.ssh/authorized-keys` file.
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
Success! Make sure to test all keypairs you created, and backup all private keys. 

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
  IdentityFile ~/.ssh/openwrt_yubi5_nano
  User root
  Port 22
### The Remote Host FreeNAS 
Host freenas
  HostName 192.168.84.85
  IdentityFile ~/.ssh/freenas
  User root
  Port 22
```
Note: You can add multiple `IdentityFile` lines for multiple keys for multiple FIDO2 devices. Save (CTRL+O, ENTER) and exit (CTRL+X). Test the config file:
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
Stop the LUCI web interface, it is not required for your router to perform its functions. 
```
root@OpenWrt:~# /etc/init.d/uhttpd stop
```
If you want to change router settings, then SSH in and start the web-ui, make your adjustments, then stop the web-ui:
```
root@OpenWrt:~# /etc/init.d/uhttpd start 
```
Advanced option: Leave the LUCI web interface on all the time, just make it accessible only thru a SSH tunnel: [here](https://openwrt.org/docs/guide-user/luci/luci.secure)

### Step 6: Disable SSH password authentication on FreeNAS
Login to your freenas web-ui. Click "accounts", "users", "root", "edit". Under "Disable Password", select "Yes". Click "Save". Now try a password based login:
```
User@Desktop ~ $ ssh root@192.168.84.85
root@192.168.84.85: Permission denied (publickey).
```
Success!

Next: [ [Bastion](4_bastion.md) ] >>
