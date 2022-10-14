[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [nginx](4_apache.md) ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ **reverseproxy** ] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

Create your reverse proxy jail as outlined in the wordpress guide [here](https://github.com/seth586/guides/blob/master/FreeNAS/webserver/6_reverse_proxy.md)

Enter your `reverseproxy` jail and add the following file, replace `mydomain` with yours and `proxy_pass` IP with `nextcloud` jail IP:
```
# nano /usr/local/etc/nginx/vdomains/domain.tld.conf
```
```
server {
        listen 443 ssl http2;

        server_name cloud.mydomain.com;
        access_log /var/log/nginx/cloud.access.log main;
        #server_tokens off;

        include snippets/mydomain.com.cert.conf;
        include snippets/ssl-params.conf;

        location / {
                include snippets/proxy-params.conf;
                proxy_pass http://192.168.84.73;
                client_max_body_size 2G;
        }
        location = /.well-known/carddav {
        return 301 https://cloud.mydomain.com/remote.php/dav;
        }
        location = /.well-known/caldav {
        return 301 https://cloud.mydomain.com/remote.php/dav;
        }
}

```
Save (CTRL+O, ENTER) and exit (CTRL+X)

Restart nginx `service nginx restart`

### nextcloud jail configuration

Exit the `reverseproxy` jail and ssh into your `nextcloud` jail. Set the jail ip of your `reverseproxy`:
```
su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set trusted_proxies --value="192.168.84.8"'
```
Change the trusted domain to your `cloud.mydomain.com`:
` su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set trusted_domains --value="cloud.mydomain.com"'`
