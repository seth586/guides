## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - **[PHP]** - [ [wordpress](5_wordpress.md) ] - [ [reverse proxy](6_reverse_proxy.md) ]

PHP is a programming language designed for interactive web content. Numerous PHP modules exist to increase the capability of this language. These PHP modules can be individually installed depending on what your plugins and themes require and isntalled with seperate packages. To see what modules are activated, type `php -m`.

Numerous modern wordpress themes rely on an import and export wordpress plugin that has not been updated in several years ([here](https://github.com/humanmade/WordPress-Importer) and [here](https://github.com/awesomemotive/one-click-demo-import)), so we are going to compile a few required modules directly into the PHP executable, providing support for this important wordpress function. 

## Configure ports Makefile
```
# portsnap fetch update
# portsnap extract
# cd /usr/ports/lang/php74
# nano Makefile
```
Add the following lines to `configure_args+=`
```
--enable-dom \
--enable-xmlreader \
```
Save (CTRL+O,ENTER) and exit (CTRL+X)

## Compile PHP with xmlreader and dom modules built in
```
# make install clean BATCH=yes
```
This will take some time. 

## Configure PHP
```
# cp /usr/local/etc/php.ini{-production,}
# nano /usr/local/etc/php.ini
```

Search (CTRL+W) for `cgi.fix_pathinfo=` and change the value to `cgi.fix_pathinfo=0`

Search (CTRL+W) for `upload_max_filesize` and change the value to something bigger. I use `upload_max_filesize = 8M`. This will be the maximum file upload size to your web server. Plugins that have to be naually uploaded can be several magabytes in size, so this value may have to change. When you create blog posts, or allow users to upload data, this is useful to restricting bandwidth and storage use.

Save (CTRL+O, ENTER) and exit (CTRL+X)
