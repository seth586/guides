
[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [postgresql](2_postgresql.md) ] - [ [synapse](3_synapse.md) ] - [ **reverse proxy** ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Reverse Proxy

Enter your `reverseproxy` jail and add the following file:
```
# nano /usr/local/etc/nginx/vdomains/domain.tld.conf
```

Remember to replace `domain.tld` with your own domain and onion.onion with your hidden service address for clients:
```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    server_name domain.tld;

        include snippets/domain.tld.cert.conf;
        include snippets/ssl-params.conf;
        
    # Token registration
    location ~ ^/(static|register) {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://192.168.84.79:5000;
    }

    location ~ ^/(_matrix|_synapse/client)/ {
        proxy_pass http://192.168.84.79:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        # Nginx by default only allows file uploads up to 1M in size
        # Increase client_max_body_size to match max_upload_size defined in homeserver.yaml
        client_max_body_size 50M;
    }

    location /.well-known/matrix/server {
      return 200 '{"m.server": "domain.tld:443"}';
      default_type application/json;
    }

    location /.well-known/matrix/client {
      return 200 '{"m.homeserver": {"base_url": "https://domain.tld"},"m.identity_server": {"base_url": "https://vector.im"}}';
      default_type application/json;
      add_header "Access-Control-Allow-Origin" *;
    }

}

# TOR HIDDEN SERVICE FOR CLIENTS 
server {
    listen 80 http2;
    
    server_name onion.onion;

    # Token registration
    location ~ ^/(static|register) {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://192.168.84.79:5000;
    }

    location /_synapse/client {
        proxy_pass http://192.168.84.79:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        client_max_body_size 50M;
    }

    location /.well-known/matrix/client {
      return 200 '{"m.homeserver": {"base_url": "http://onion.onion"},"m.identity_server": {"base_url": "https://vector.im"}}';
      default_type application/json;
      add_header "Access-Control-Allow-Origin" *;
    }

}
```
Save (CTRL+o, ENTER) and exit (CTRL+x)

Restart nginx
```
# service nginx restart
```
