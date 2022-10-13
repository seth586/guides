[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [nginx](4_apache.md) ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ **nextcloud** ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install Nextcloud
```
# cd /tmp
# wget https://download.nextcloud.com/server/releases/latest.tar.bz2
# wget https://download.nextcloud.com/server/releases/latest.tar.bz2.sha512
# shasum -a 512 -c latest.tar.bz2.sha512
# tar -xf latest.tar.bz2 -C /usr/local/www
# chown -R www:www /usr/local/www/nextcloud
```

### Nextcloud: enable redis cacheing
```
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set redis host --value="/var/run/redis/redis.sock"'
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set redis port --value=0 --type=integer'
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set memcache.local --value="\OC\Memcache\Redis"'
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set memcache.locking --value="\OC\Memcache\Redis"'
# service nginx restart
```

### Configure apache
Find the directory block below and edit `nano /usr/local/etc/apache24/httpd.conf`:
```
DocumentRoot "/usr/local/www/nextcloud"
<Directory "/usr/local/www/nextcloud">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    #
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
    Options Indexes FollowSymLinks

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   AllowOverride FileInfo AuthConfig Limit
    #
    AllowOverride All

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>
```
Save (CTRL+O, ENTER) and exit (CTRL+X)

`nano /usr/local/etc/apache24/Includes/cloud.mydomain.com.conf`:
```
<VirtualHost *:80>
    DocumentRoot "/usr/local/www/nextcloud"
    ServerName cloud.mydomain.com
    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9000/"
    </FilesMatch>
    DirectoryIndex /index.php index.php
</VirtualHost>
```

