[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [nginx](4_apache.md) ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ **collabora** ]

## Guide to Nextcloud server on TrueNAS

Spin up ubuntu VM

https://docs.nextcloud.com/server/latest/admin_manual/office/example-ubuntu.html
https://www.collaboraoffice.com/code/linux-packages/

```
sudo apt update -y
sudo apt install coolwsd code-brand -y
sudo systemctl start coolwsd && sudo systemctl enable coolwsd
sudo systemctl status coolwsd

sudo coolconfig set ssl.enable false
sudo coolconfig set ssl.termination true
sudo coolconfig set storage.wopi.host cloud.example.com
sudo coolconfig set-admin-password
sudo systemctl restart coolwsd

sudo netstat -tunlp | grep 9980
```
