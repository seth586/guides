## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - [ [PHP](4_php.md) ] - [ [wordpress](5_wordpress.md) ] - **[reverse proxy]**


work in progress

Use this for now

https://www.samueldowling.com/2020/01/18/nginx-reverse-proxy-freenas-ssl-tls/


Create a new jail with a static IP address outside the range of yoru routers DHCP IP range. The default DHCP range on openwrt is 192.168.0.100 thru 192.168.0.254, I will use 192.168.84.44 as an example.

