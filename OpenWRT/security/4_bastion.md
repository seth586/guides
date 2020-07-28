[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - [ [Generate Keys](3_keys.md) ] - **[Bastion]**

## SSH Bastion Security Hardening Guide
### Bastion
`User@Desktop ~ $ nano ~/.ssh/config` on client to ProxyJump thru our router bastion:
```
### The Bastion Host
Host openwrt
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
### Configure FreeNAS to only accept connecitons thru the bastion
Log in to the freenas web-ui, click "services", and click the "configure" icon on the SSH line. Click "advanced mode". Add the following line to "extra options":
```
AllowUsers root@192.168.84.1
```
FreeNAS will now only accept incoming SSH requests from your openwrt bastion, effectively requiring a sucessful ssh login to your router first! 

If your router goes up in flames, no worries. Your router is not storing any private keys! A new router with the same IP address configuration, your backup private keys & FIDO2 devices are all you need to SSH back in to freenas. 

