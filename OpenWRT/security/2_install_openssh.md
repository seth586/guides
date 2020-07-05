[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - **[Install OpenSSH on OpenWRT]** - [ [Generate Keys](3_keys.md) ] - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### Install OpenSSH on OpenWRT
By default OpenWRT comes installed with a lite SSH server called `dropbear`. However if we want to use more advanced features like FIDO/U2F, we will need the heavier & full featured `OpenSSH`. Don't worry, if you're running a modern OpenWRT enabled router you'll be just fine!
