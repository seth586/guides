## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - [ [PHP](4_php.md) ] - [ [wordpress](5_wordpress.md) ] - **[reverse proxy]**

A reverse proxy allows you to host multiple websites from the same IP address. Our reverse proxy will also offer SSL/TLS termination, such as wildcard.sampledomain.com or sampledomain.com.

work in progress

Use this for now

https://www.samueldowling.com/2020/01/18/nginx-reverse-proxy-freenas-ssl-tls/


## Create a new jail
Login to the TrueNAS web-ui. Create a new jail with a static IP address outside the range of your routers DHCP IP range. The default DHCP range on openwrt is 192.168.0.100 thru 192.168.0.254, I will use 192.168.84.44 as an example.

![ReverseProxyJail](images/reverseproxyjail.png)

## Port Forwarding
Forward ports 443 and 80 to your reverseproxyjail in your router.

![ReverseProxyPortForwardRouter](images/reverseproxyportforwardrouter.png)

## Install nginx
SSH into TrueNAS and switch to your reverseproxy jail.
```
# iocage console reverseproxy
# pkg update
# pkg install nginx nano python
# sysrc nginx_enable=yes
```

## Create AWS Route 53 user with minimum permissions

In this example we will use certbot with the Amazon Route 53 plugin for our domain name server. For other dns providers using this method run `pkg search certbot` to see a list of provider plugins. This list is not exhaustive, there are many different ways to obtain and renew SSL/TLS certificates, refer to documentation provided by your domain provider.

At this point you should have sucessfully registered a domain on Amazon's Route 53. Login to your Amazon Web Services as a root user. Bring up the "services" menu on the top left, and under "Security, Identity, & Complaince" click on IAM to bring up the "Identity and Access Management" dashboard. Click "Policies" on the left menu, then click "Create Policy". We can build the permissions thru the visual editor:

Under "Service" select "Route 53"
Under "Actions" select the following permissions required by [documentation for certbot's route 53 plugin](https://certbot-dns-route53.readthedocs.io/en/stable/):  

route53:ListHostedZones
route53:GetChange
route53:ChangeResourceRecordSets

Under Resources select "All resources". Click "Review Policy". Give the policy a name, I used `certbot`. Write a description to remind you what its for: "Policy so my reverse proxy can request and renew SSL and TLS certificates". Click "Create policy". You should now have the "certbot" policy in your "Policies" list.

Now lets create a user with this policy. Click "Users" on the left menu. Click "Add user". Lets call it `reverseproxy`. Under "access type*" select "Programmatic access". Click "Next: Permissions". Click "Attach existing policies directly" and search for your "certbot" policy. Click the checkmark to the left of "certbot". Click "Next: Tags". Click "Next: Review". Click "Create User". Write down the Access key ID and Secret access key, and store this information in a safe place. Click "close". You should now see the user "reverse proxy" listed in your user list.

## Configure certbot to request and renew SSL/TLS certificates
```
# pkg install py37-certbot openssl py37-certbot-dns-route53 awscli
# aws configure
```
Answer the 4 questions:
AWS Access Key ID
AWS Secret Access Key
Default region name (4 options, on your aws console select the "global" menu to the right of your username. As of writing region options are US East (N. Virginia) `us-east-1`
US East (Ohio)`us-east-2`
US West (N. California)`us-west-1`
US West (Oregon)`us-west-2`


