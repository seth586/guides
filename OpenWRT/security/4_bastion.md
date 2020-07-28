[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - [ [Generate Keys](3_keys.md) ] - **[Bastion]**

## SSH Bastion Security Hardening Guide
### Bastion
`User@Desktop ~ $ nano ~/.ssh/config` on client to ProxyJump thru our router bastion:
```
### The Bastion Host
Host router
  HostName 192.168.84.1
  IdentityFile ~/.ssh/openwrt
  User root
  Port 22
### The Remote Host
Host freenas
  HostName 192.168.84.85
  IdentityFile ~/.ssh/openwrt
  User root
  ProxyJump openwrt
```
