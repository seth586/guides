## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - [ [PHP](4_php.md) ] - [ [wordpress](5_wordpress.md) ] - **[reverse proxy]**

A reverse proxy allows you to host multiple websites from the same IP address. Our reverse proxy will also offer SSL/TLS termination, such as wildcard.sampledomain.com or sampledomain.com.

## Create a new jail
Login to the TrueNAS web-ui. Create a new jail with a static IP address outside the range of your routers DHCP IP range. The default DHCP range on openwrt is 192.168.0.100 thru 192.168.0.254, I will use 192.168.84.44 as an example.

![ReverseProxyJail](images/reverseproxyjail.png)

## Router Configuration
Forward ports 443 and 80 to your reverseproxyjail in your router.

![ReverseProxyPortForwardRouter](images/reverseproxyportforwardrouter.png)

Create hostnames:

Click on "Network" -> "Hostnames"

Create an entry for every domain and subdomain you are going to set up so you can access the domain from inside your LAN. Have it resolve to your reverse proxy:

![RouterHostname](images/routerhostname.png)

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

route53:ListHostedZones route53:GetChange route53:ChangeResourceRecordSets

Under Resources select "All resources". Click "Review Policy". Give the policy a name, I used `certbot`. Write a description to remind you what its for: "Policy so my reverse proxy can request and renew SSL and TLS certificates". Click "Create policy". You should now have the "certbot" policy in your "Policies" list.

Now lets create a user with this policy. Click "Users" on the left menu. Click "Add user". Lets call it `reverseproxy`. Under "access type*" select "Programmatic access". Click "Next: Permissions". Click "Attach existing policies directly" and search for your "certbot" policy. Click the checkmark to the left of "certbot". Click "Next: Tags". Click "Next: Review". Click "Create User". Write down the Access key ID and Secret access key, and store this information in a safe place. Click "close". You should now see the user "reverse proxy" listed in your user list.

## Configure certbot to request and renew SSL/TLS certificates
```
# pkg install py37-certbot openssl py37-certbot-dns-route53 awscli
# aws configure
```
Answer the 4 questions:

AWS Access Key ID : `insert access key id for your reverseproxy aws user`

AWS Secret Access Key : `insert the secret access key for your reverseproxy aws user`

Default region name :

On your aws console select the "global" menu to the right of your username. As of writing region options are 

US East (N. Virginia) `us-east-1`

US East (Ohio)`us-east-2`

US West (N. California)`us-west-1`

US West (Oregon)`us-west-2`

Default output format: `text`

## Request domain and wildcard certificate

Make sure to replace "example.com" with your own domain name!
```
certbot certonly --dns-route53 -d 'example.com,*.example.com'
```

## Configure certificate auto-renewal

```
# setenv EDITOR nano
# crontab -e
```
Add the following line:
```
0 0,12 * * * /usr/local/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/local/bin/certbot renew --quiet --deploy-hook "/usr/sbin/service nginx reload"
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

## Configure NGINX
These are the following configuration files we are going to create: This simplifies the addition and removal of domains that you choose to host:
```
/usr/local/etc/nginx/nginx.conf
/usr/local/etc/nginx/vdomains/subdomain1.example.com.conf
/usr/local/etc/nginx/vdomains/subdomain2.example.com.conf
/usr/local/etc/nginx/snippets/example.com.cert.conf
/usr/local/etc/nginx/snippets/ssl-params.conf
/usr/local/etc/nginx/snippets/proxy-params.conf
/usr/local/etc/nginx/snippets/internal-access-rules.conf
```
### Certificate Configuration:
This file details the location of your certificates. Create one per domain.
```
# mkdir /usr/local/etc/nginx/snippets
# nano /usr/local/etc/nginx/snippets/example.com.cert.conf
```
Add the following lines. Remember to replace `example.com` with your domain:
```
# certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
ssl_certificate /usr/local/etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key /usr/local/etc/letsencrypt/live/example.com/privkey.pem;

# verify chain of trust of OCSP response using Root CA and Intermediate certs
ssl_trusted_certificate /usr/local/etc/letsencrypt/live/example.com/chain.pem;
```
Save (CTRL+O, ENTER) and exit (CTRL+X)
### SSL Configuration:

```
# curl https://ssl-config.mozilla.org/ffdhe2048.txt > /usr/local/etc/ssl/dhparam.pem
# nano /usr/local/etc/nginx/snippets/ssl-params.conf
```
You can use https://ssl-config.mozilla.org/ to create the parameters, or just copy paste below:
```
ssl_session_timeout 1d;
ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
ssl_session_tickets off;

# curl https://ssl-config.mozilla.org/ffdhe2048.txt > /usr/local/etc/ssl/dhparam.pem
ssl_dhparam /usr/local/etc/ssl/dhparam.pem;

# intermediate configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;

# HSTS (ngx_http_headers_module is required) (63072000 seconds)
add_header Strict-Transport-Security "max-age=63072000" always;

# OCSP stapling
ssl_stapling on;
ssl_stapling_verify on;

# replace with the IP address of your resolver
resolver 192.168.0.1;
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

### Proxy Header Configuration
```
# nano /usr/local/etc/nginx/snippets/proxy-params.conf
```
Paste the following:
```
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host $server_name;
proxy_set_header X-Forwarded-Ssl on;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_http_version 1.1;
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

### Access policy configuration
This is the policy that we’ll apply to services that you don’t want to be externally available, but still want to access it using HTTPS on your LAN.
```
# nano /usr/local/etc/nginx/snippets/internal-access-rules.conf
```
Paste the following. Make sure you select your appropriate LAN subnet range:
```
allow 192.168.0.0/24;
deny all;
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

### Virtural domain configuration
```
# mkdir /usr/local/etc/nginx/vdomains
```
This directory will contain the configurations for each domain and their subdomains you wiosh to proxy to. Create one configuration file for each domain (such as https://example.com) and/or subdomain (such as https://subdomain.example.com

Lets create a proxy for "https://blog.example.com"
```
# nano /usr/local/etc/nginx/vdomains/blog.example.com.conf
```
Paste the following:
```
server {
        listen 443 ssl http2;

        server_name blog.example.com;
        access_log /var/log/nginx/blog.access.log;
        error_log /var/log/nginx/blog.error.log;

        include snippets/example.com.cert.conf;
        include snippets/ssl-params.conf;

        location / {
                include snippets/proxy-params.conf;
                # Uncomment below if you only want internal access on your LAN
                # include snippets/internal-access-rules.conf;
                proxy_pass http://192.168.0.10;
        }
}
```
Save (CTRL+O, ENTER) and exit (CTRL+X).

Notice how this configuration recalls our other created config files. Nice and clean!

## NGINX.conf
Now lets tie it all together!

```
# mv /usr/local/etc/nginx/nginx.conf /usr/local/etc/nginx/nginx.conf.bak
# nano /usr/local/etc/nginx/nginx.conf
```
Paste the following.
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;

    # Redirect all HTTP traffic to HTTPS
    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        return 301 https://$host$request_uri;
    }

    # Import server blocks for all subdomains
    include "vdomains/*.conf";
}
```
Save (CTRL+O, ENTER), exit (CTRL+X) and restart nginx `service nginx restart`.

### Credits
The samueldowling.com blog!
