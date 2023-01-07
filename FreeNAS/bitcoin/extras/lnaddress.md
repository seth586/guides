
Create LNADDRESS jail, ssh in

`pkg install nginx yarn-node18 git nano`

### Install ligess
```
# git clone https://github.com/dolu89/ligess
# cd ligess && yarn install
# cp .env.example .env
# nano .env
```
Convert `invoice.macaroon` to hex in your bitcoin jail:

Edit .env:
```

```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

Test:
```
# yarn dev
```

### Set up nginx
```
# sysrc nginx_enable=yes
# nano /usr/local/etc/nginx/nginx.conf
```
Add the following location block to your server block:
```
location /.well-known/lnurlp/seth586 {
        proxy_pass http://127.0.0.1:3000;
}
```
Save (CTRL+O, ENTER) and Exit (CTRL+X)

Start nginx ` service nginx start`
