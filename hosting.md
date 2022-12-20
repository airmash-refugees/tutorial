# Server

- Rent a server
- get a domain name and modify DNS records to point to your server: create a www record and a ffa record.
- install nginx and nodejs 12 and npm
- use letsencrypt certbot to create the certificate files
- git clone ab-server into it, copy .env.production to .env, and change some settings:

```
HOST=<your public ip address>
ENDPOINTS_TLS=true
BOTS_IP="127.0.0.1,<your public ip address>"
``` 

- copy the three required cert files to ab-server/certs from the location where letsencrypt stores them. 
The dhparam file may be on a different location and have a different name: /etc/letsencrypt/ssl-dhparams.pem
- build ab-server and start it (docker does not work at this point, dec-22)

create /etc/nginx/sites-enabled/ffa.domainname.conf. Create listen 80 without ssl first, and let certbot create the certs for you. Final config:

```nginx
server {
        listen 443 ssl;
        server_name ffa.subdomain.name.here;
        ssl_certificate /path/to/cert.pem;
        ssl_certificate_key /path/to/privkey.pem;

        location / {
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $host;

                proxy_pass https://ws-backend;

                proxy_ssl_certificate     /path/to/cert.pem;
                proxy_ssl_certificate_key /path/to/privkey.pem;

                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }
  }

upstream ws-backend {
        ip_hash;

        server ffa.subdomain.name.here:3501;
}
```

# Bots

At this point, there is no simple out-of-the-box way to run bots afaik. Working on it.

# Frontend

Hosting a frontend is not required, but is just 
- clone airmash-frontend
- npm run build
- create /etc/nginx/sites-enabled/www-domainname.conf

```nginx
server {
        server_name www.subdomainname.here;
        root /path/to/airmash-frontend/dist;
        index index.html;

        listen 443 ssl; # managed by Certbot
        ssl_certificate /path/to/fullchain.pem; # managed by Certbot
        ssl_certificate_key /path/to/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = www.subdomain.here) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name www.subdomain.here;
    return 404; # managed by Certbot
}
```

# publish the server
- change the server list in the games repo
