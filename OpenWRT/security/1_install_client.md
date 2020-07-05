[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - **[OpenSSH Client]** - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - [ [Generate Keys](3_keys.md) ] - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### OpenSSH Clients
Before we begin changing our servers, we need to make sure our clients have up to date versions of OpenSSH to utilize the full featureset of security in this guide.
To utilize the most secure cryptography, OpenSSH version 6.5 or newer is required. To utilize a FIDO/U2F device, such as yubikey, you need a minimum OpenSSH version 8.1 on the server and 8.2 on the client.

#### Open SSH Client - Windows
As of writing, Putty does not support the interactive ed25519-sk signature types, so if you wan't to use a FIDO/U2F device such as the yubikey, you will need something else. For now I recommend `cygwin64`, download on [www.cygwin.com](https://www.cygwin.com)

The `cygwin64` installer is where you add or remove pre-compiled packages. Use default installation options. When you get to the "select a mirror" page, click any mirror you wish to use, then press next. You will be greeted with a "select Packages" window. Under "view", click "full". Search for `OpenSSH`. Under the "New" column, select the OpenSSH version you would like to install, make sure to use version 8.2 or newer.

Now search for `nano`, and under the "New" column, select the latest version. Finish the installation with default options. The installation will create a "Cygwin64 Terminal" icon on your desktop. To read your FIDO/U2F USB device, we will need to run `cygwin64` as an administrator. Right click on the shortcut, select "Properties", click the "Compatibility" tab, and select the "Run this program as an administrator" box. Click "OK".

Now launch `cygwin64`, and check the OpenSSH version installed:
```
$ ssh -V
OpenSSH_8.3p1, OpenSSL 1.1.1f  31 Mar 2020
```

Success!

#### OpenSSH Client - Linux


Next: [ [OpenSSH on OpenWRT](2_install_openssh.md) ] >>
