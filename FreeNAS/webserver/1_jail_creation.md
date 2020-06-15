## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - **[Jail Creation]** - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - [ [PHP](4_php.md) ] - [ [wordpress](5_wordpress.md) ]  - [ [reverse proxy](6_reverse_proxy.md) ]

Login to the TrueNAS web-ui. Create a new jail with a static IP address outside the range of your router's DHCP IP range. The default DHCP range on openwrt is 192.168.0.100 thru 192.168.0.254, I will use 192.168.84.80 as an example and call the jail 'blog'.

## SSH into your
SSH into TrueNAS and switch to your blog jail.
```
# iocage console blog
```

Next: [ [nginx](2_nginx.md) ] >>
