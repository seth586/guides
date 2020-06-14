**[Intro]** - [ [Jail Creation](1_jail_creation.md) ]

As commercial, government & social pressures for censorship continue to increase, self sovreignity on the internet is becoming more and more improtant. This guide will help you configure a wordpress webserver on FreeNAS & TrueNAS flavors of FreeBSD. The internet is still relatively permissioned around centralized domain name service providers, so make sure to select a provider that will not censor you based on your character, beliefs, or content. 

Wordpress is the single most popular web structure online. Countless free and premium plugins and themes are available to start your own user interactive blog, business, resume, you name it. When you are finished with this guide, you will have a working wordpress website hosted in an iocage jail, with a seperate reverse proxy jail that redirects domain requests from your home IP to the approprite jail, allowing you to host multiple jails. Do be aware that using a domain directed to your home IP address will still dox your location. If you can not dox your location, consider using a VPS tor redirect or a tor only address for your website.

Your server is only secure as its firewall. For home use I highly recommend using a capable router running OpenWRT.
