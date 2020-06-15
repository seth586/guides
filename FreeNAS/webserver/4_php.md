## Guide to a self hosted wordpress website on FreeNAS/TrueNAS
[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ [nginx](2_nginx.md) ] - [ [mysql](3_mysql.md) ] - **[PHP]** - [ [wordpress](5_wordpress.md) ] - [ [reverse proxy](6_reverse_proxy.md) ]

PHP is a programming language designed for interactive web content. Numerous PHP modules exist to increase the capability of this language. These PHP modules can be selected depending on what your plugins and themes require and isntalled with seperate packages.

Numerous modern wordpress themes rely on an import and export wordpress plugin that has not been updated in several years ([here](https://github.com/humanmade/WordPress-Importer) and [here](https://github.com/awesomemotive/one-click-demo-import)), so we are going to compile a few required modules directly into the PHP executable, providing support for this important wordpress function. 
