# Digisol website

Create folder on server:

```bash
mkdir /var/www/digisol.agency
```

Create environment variables file:

```bash
cd /var/www/digisol.agency
touch .env
echo "NEXT_PUBLIC_API_URL=https://digisol.agency" > .env
```

Upload source files into that folder.

Install all dependencies and compile next js app:

```bash
yarn install
yarn build
```

Create systemd configuration file:

```bash
[Unit]
Description=Digisol Agency
After=network.target

[Service]
WorkingDirectory=/var/www/digisol.agency/
ExecStart=/root/.nvm/versions/node/v16.17.0/bin/node /var/www/digisol.agency/node_modules/next/dist/bin/next start -p 3007
EnvironmentFile=/var/www/digisol.agency/.env

Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=live_digisol
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

>NOTE: node js root path can be different. Depends on how you install nodejs - with `nvm` or using `apt install` or any other way. Make sure that node path are actualy exists.

Repository uses Github Action configurations files to automatic deploy changes on server:
https://github.com/DigisolAgency/new_digisol_website/tree/master/.github/workflows

They trigger deploy actions on push into master ( for production website ) and on push into dev ( for development website ). Development website uses same configurations on server excepts path for source code is `/var/www/dev.digisol.agency/`.

### NGINX

```bash
upstream live_digisol_website {
    server 127.0.0.1:3007;
    keepalive 64;
}

server {
    server_name digisol.agency;

    access_log /var/log/nginx/dev.digisol.agency.access.log;
    error_log  /var/log/nginx/dev.digisol.agency.error.log;

    root /var/www/digisol.agency;

    location /_next/static/ {
        alias /var/www/digisol.agency/.next/static/;
    }

    location / {
        root /var/www/digisol.agency;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_pass http://live_digisol_website;
        proxy_redirect off;
        proxy_read_timeout 240s;
    }
}
```
