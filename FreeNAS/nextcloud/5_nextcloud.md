[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [apache](4_apache.md) ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ **nextcloud** ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Install Nextcloud
```
# cd /tmp
# wget https://download.nextcloud.com/server/releases/latest.tar.bz2
# wget https://download.nextcloud.com/server/releases/latest.tar.bz2.sha512
# shasum -a 512 -c latest.tar.bz2.sha512
# tar -xf latest.tar.bz2 -C /usr/local/www
# rm latest.tar.bz2
# rm latest.tar.bz2.sha512
# chown -R www:www /usr/local/www/nextcloud
```

### Configure apache
`nano /usr/local/etc/apache24/httpd.conf` and edit the directory block below:
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
        SetHandler "proxy:unix:/var/run/php-fpm.sock|fcgi://localhost"
    </FilesMatch>
    DirectoryIndex /index.php index.php
</VirtualHost>
```
Restart apache `service apache24 restart`

Navigate to your jail IP and configure nextcloud.

Create an admin `username` and `password`. 

datafolder = `/mnt/data`

Database user = `nextcloud`

Database password = `your_database_password`

Database name = `nextcloud`

Database host = `localhost:/var/run/mysql/mysql.sock`

### Nextcloud: enable redis cacheing
```
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set redis host --value="/var/run/redis/redis.sock"'
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set redis port --value=0 --type=integer'
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set memcache.local --value="\OC\Memcache\Redis"'
# su -m www -c 'php /usr/local/www/nextcloud/occ config:system:set memcache.locking --value="\OC\Memcache\Redis"'
# service apache24 restart
```

### Optional: shell alias for occ
typing in `su -m www -c 'php /usr/local/www/nextcloud/occ command'` sucks, lets make a shell alias so we can run occ globally:
```
nano /root/.cshrc
```
Add the alias:
```
alias occ       'su -m www -c '\''php /usr/local/www/nextcloud/occ "$1"'\'''
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

Refresh your shell and try it out:
```
source ~/.cshrc
occ status
```

