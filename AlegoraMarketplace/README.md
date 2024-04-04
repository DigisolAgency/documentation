# Alegora Marketplace

Create folder on server:

```bash
mkdir /var/www/alegora-marketplace
```

Upload source files into that folder.

Install all dependencies and compile next js app:

```bash
cd /var/www/alegora-marketplace

yarn install
yarn build
```

Create systemd configuration file:

```bash
[Unit]
Description=Alegora Marketplace
After=network.target

[Service]
WorkingDirectory=/var/www/alegora-marketplace/
ExecStart=/root/.nvm/versions/node/v20.11.1/bin/node /var/www/alegora-marketplace/node_modules/next/dist/bin/next start -p 3099

Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=alegora_marketplace
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

>NOTE: node js root path can be different. Depends on how you install nodejs - with `nvm` or using `apt install` or any other way. Make sure that node path are actualy exists.

Repository uses Github Action configurations file to automatic deploy changes on server:
https://github.com/DigisolAgency/alegora-marketplace/blob/main/.github/workflows/deploy.yml

They trigger deploy actions on push into master ( for production website ).

### NGINX

```bash
upstream alegora_marketplace_worker {
    server 127.0.0.1:3099;
    keepalive 64;
}

server {
    server_name alegora-marketplace.digisol.agency;
    access_log /var/log/nginx/alegora.digisol.agency.access.log;
    error_log  /var/log/nginx/alegora.digisol.agency.error.log;
    root /var/www/alegora-marketplace;
    location /_next/static/ {
        alias /var/www/alegora-marketplace/.next/static/;
    }
    
    location / {
        root /var/www/alegora-marketplace;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
    
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        proxy_pass http://alegora_marketplace_worker;
        proxy_redirect off;
        proxy_read_timeout 240s; 
    }   
}
```

Next create symlink for to be able nginx serve new domain:
```bash
ln -s /etc/nginx/sites-available/alegora-marketplace.digisol.agency /etc/nginx/sites-enabled
```

Next create ssl certificate using python certbot:

```bash
certbot --nginx -d alegora-marketplace.digisol.agenc --force-renew --redirect
```

Make sure you "point" server IP for this subdomain on domain DNS settings.

Next restart nginx:

```bash
# check if everything is ok
nginx -t

systemctl restart nginx
```