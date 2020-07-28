[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - **[OpenSSH Client]** - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - [ [Generate Keys](3_keys.md) ] - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### OpenSSH Clients
Before we begin changing our servers, we need to make sure our clients have up to date versions of OpenSSH to utilize the full featureset of security in this guide.
To utilize the most modern cryptographic function ed25519, OpenSSH version 6.5 or newer is required. To utilize a FIDO/U2F device (such as yubikey, trezor or ledger nano) you need a minimum OpenSSH version 8.1 on the server and 8.2 on the client to support ed25519-sk interactive signature types.

#### Open SSH Client - Windows
As of writing, Putty does not support the interactive ed25519-sk signature types, so if you wan't to use a FIDO/U2F device, you will need something else. For now I recommend `cygwin64`, download on [www.cygwin.com](https://www.cygwin.com)

The `cygwin64` installer is where you add or remove pre-compiled packages. Use default installation options. When you get to the "select a mirror" page, click any mirror you wish to use, then press next. You will be greeted with a "select Packages" window. Under "view", click "full". Search for `OpenSSH`. Under the "New" column, select the OpenSSH version you would like to install, make sure to use version 8.2 or newer.

Now search for `nano`, and under the "New" column, select the latest version. Finish the installation with default options. The installation will create a "Cygwin64 Terminal" shortcut on your desktop. To read your FIDO/U2F USB device, we will need to run `cygwin64` as an administrator. Right click on the shortcut, select "Properties", click the "Compatibility" tab, and select the "Run this program as an administrator" box. Click "OK".

Now launch `cygwin64`, and check the OpenSSH version installed:
```
User@Desktop ~ $ ssh -V
OpenSSH_8.3p1, OpenSSL 1.1.1f  31 Mar 2020
```

Success! Now verify your current SSH session works, replace `192.168.84.85` with your FreeNAS' local IP address:
```
User@Desktop ~ $ ssh root@192.168.84.85
```
Type in your password, you should be able to sucessfully SSH in using password based authentication.

#### OpenSSH Client - Linux
```
User@Desktop ~ $ sudo apt-get update
User@Desktop ~ $ sudo apt-get install openssh
User@Desktop ~ $ ssh -V
OpenSSH_8.3p1, OpenSSL 1.1.1g, 21 Apr 2020
```

#### OpenSSH Client - Android

I don't believe interactive FIDO/U2F works with android, but you can still securely log in using a password protected public/private key authentication! Download Termux at [https://termux.com/](https://termux.com/)

```
$ pkg install openssh
$ ssh -V
OpenSSH_8.3p1, OpenSSL 1.1.1g, 21 Apr 2020
```

#### OpenSSH Client - Mac

You're on your own, buddy! Maybe start your search at https://brew.sh/

Next: [ [OpenSSH on OpenWRT](2_install_openssh.md) ] >>
