---
title: Instalando Shop Juice
tags:
    - seguridad
    - configuraciones 
---

## Introducción
[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) es una aplicación creada deliberadamente insegura, para práctica

## Entorno previo
Es un sistema Debian 12, donde ya habían otras aplicaciones de prueba en PHP y se usaba nginx. Así que tuve que instalar node, npm y pm2:

```bash
apt install nodejs
apt install npm --no-install-suggests --no-install-recommends
npm install pm2 -g
```

## Instalación
El `npm install` realmente es un largo proceso, no solo por la descarga de paquetes, sino también por todas las tareas post-instalación

```bash
cd /var/www/
git clone https://github.com/juice-shop/juice-shop.git --depth 1
cd juice-shop
npm install
```

Seguimos configurando `pm2`, creo que el proceso va más o menos así (Para variar, lo hice un poco más confuso). Desde el directorio `/var/www/juice-shop`:

```bash
pm2 startup
systemctl enable --now pm2-root
pm2 start npm --name "juice-shop" -- start
pm2 save
```

Por último, la configuración en Nginx va de la siguiente manera:
```nginx
upstream juice_shop {
    server 127.0.0.1:3000;
    keepalive 64;
}

server {
    listen 80;
    
    server_name juice-shop.sanidad.gob.sv;
   
    location / {
    	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
    	proxy_set_header Host $http_host;
        
    	proxy_http_version 1.1;
    	proxy_set_header Upgrade $http_upgrade;
    	proxy_set_header Connection "upgrade";
        
    	proxy_pass http://juice_shop/;
    	proxy_redirect off;
    	proxy_read_timeout 240s;
    }
}
```

Reiniciamos nginx y resolvemos como hacer para resolver esa dirección

## Fuentes
* [Building Your Self-Hosted Hacking Lab - Part 2](https://albertlacasta.com/build-selfhosted-pentesing-lab-part-2/)
* [PM2 Process Management Quick Start](https://pm2.keymetrics.io/docs/usage/quick-start/)
* [Nginx as a HTTP proxy](https://pm2.keymetrics.io/docs/tutorials/pm2-nginx-production-setup)
