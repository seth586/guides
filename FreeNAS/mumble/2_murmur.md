[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail_creation.md) ] - [ **Murmur** ] - [ [SSL & Domain](3_ssl_domain.md) ] - [ [Basic ACL Config](4_acl.md) ]

## Guide to Mumble server (murmur) on FreeNAS/TrueNAS
### Murmur

SSH into TrueNAS and switch to your blog jail.
```
# iocage console mumble
```

Lets get installing the server [murmur](https://wiki.mumble.info/wiki/Running_Murmur)!
```
# pkg install -y murmur nano
# sysrc murmur_enable=yes
```

### Configure Murmur
```
# nano /usr/local/etc/murmur.ini
```
You can read up on configuration options [here](https://wiki.mumble.info/wiki/Murmur.ini). Edit the following lines:

Uncomment the autoban lines so that your server can not be "denial of service" attacked. Remove the ";" before each item:
```
autobanAttempts = 10
autobanTimeframe = 120
autobanTime = 300
```
Save (CTRL+O, ENTER) and exit (CTRL+X).

### Start the server
```
# service murmur start && tail -f /var/log/murmur/murmur.log
```
Notice the log line `<W>2020-06-30 00:06:38.398 1 => Password for 'SuperUser' set to 'cHxBymvuygMy'` that sets the SuperUser password: We will use these account credentials to configure the Access Control List, then hopefully never use the superuser account again! To exit the live logging, press CTRL+C.

### Connect a mumble client
On a client machine, download the mumble client [here](https://www.mumble.info/). Open the program and click "Add New...". For a username use `SuperUser`, for the password use 
what the server logs revealed on the previous step, such as `cHxBymvuygMy` in our example. For the address, use your local jail ip, `192.168.84.99`. Now click "connect".

You will be prompted with a self-signed certificate warning. Click "yes" for now. 

You should get a connected message. Congradulations, you're up and running!

### About that warning...

To avoid the self-signed certificate warning popup for yourself and other users in the future, we can configure a domain and SSL/TLS certificate signed by a certificate authority, which is outlined in the next step of this guide. 

You can skip the SSL/TLS step if you want, and your communications will still be end to end encrypted. Just hand out your public IP address to friends and family if you have a static IP address. Or you could even set up a static tor address to forward to your mumble jail ip and port 64738. 

Next: [ [SSL & Domain](3_ssl_domain.md) ] >>
