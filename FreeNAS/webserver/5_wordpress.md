[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - [ [PHP](4_php.md) ] - **[wordpress]** - [ [reverse proxy](6_reverse_proxy.md) ]

## Configure & Install Wordpress
Now that we have our FEMP stack up and running, lets install wordpress!
```
# cd ~
# fetch https://wordpress.org/latest.tar.gz
# tar -zxvf latest.tar.gz
# rm latest.tar.gz
# cp wordpress/wp-config-sample.php wordpress/wp-config.php
# nano wordpress/wp-config.php
```
Add the first three commented out `//` lines right above the `// ** MySQL settings` comment. We will use this later to upgrade our connection to HTTPS once the reverse proxy is working. Then change `WP_HOME` and `WP_SITEURL` to your jail IP. This will also change to your domain name once the reverse proxy is in place. Change `database_name_here`, `username_here` and `password_here` with the values you used when setting up the mysql database. Change `localhost` to the unix socket created by `mariadb-server`.
```
// if (!empty($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
//        $_SERVER['HTTPS'] = 'on';
//}
define( 'WP_HOME', 'http://192.168.84.80' );
define( 'WP_SITEURL', 'http://192.168.84.80' );
define( 'DB_NAME', 'database_name_here' );
define( 'DB_USER', 'username_here' );
define( 'DB_PASSWORD', 'password_here' );
define( 'DB_HOST', 'localhost:/tmp/mysql.sock' );
```
In a browser, navigate to https://api.wordpress.org/secret-key/1.1/salt/ . This will generate salt to create unique cookies for your site visitors. Cookies are used to remember login activity, shopping carts, etc. Copy and replace the lines generated.

Save (CTRL+O, ENTER) and exit (CTRL+X)
```
# cd ~
# cp -r wordpress/* /usr/local/www/nginx
# rm -r wordpress
# chown -R www:www /usr/local/www/nginx/
# service nginx restart
```

Navigate to your jail IP, you should be greeted with the wordpress installation! If you ever lose the login URL, its `http://192.168.84.58/wp-login.php`

Next: [ [reverse proxy](6_reverse_proxy.md) ] >>
