# Steps to build up a web with SSL
1. Levantar una web con http

mkdir -p /home/ubuntu/docker/letsencrypt-docker-nginx/src/letsencrypt/letsencrypt-site

vim /home/ubuntu/docker/letsencrypt-docker-nginx/src/letsencrypt/docker-compose.yml

```
services:

  letsencrypt-nginx-container:
    container_name: 'letsencrypt-nginx-container'
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./letsencrypt-site:/usr/share/nginx/html
    networks:
      - docker-network

networks:
  docker-network:
    driver: bridge
```

vim /home/ubuntu/docker/letsencrypt-docker-nginx/src/letsencrypt/nginx.conf

```
server {
    listen 80;
    listen [::]:80;
    server_name crespodev.com;

    location ~ /.well-known/acme-challenge {
        allow all;
        root /usr/share/nginx/html;
    }

    root /usr/share/nginx/html;
    index index.html;
}
```

vim /home/ubuntu/docker/letsencrypt-docker-nginx/src/letsencrypt/letsencrypt-site/index.html

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>Let's Encrypt First Time Cert Issue Site</title>
</head>
<body>
    <h1>This is a webpage for the certification</h1>
    <p>
        This is the temporary site that will only be used for the very first time SSL certificates are issued by Let's Encrypt's
        certbot.
    </p>
</body>
</html>
```

cd /home/ubuntu/docker/letsencrypt-docker-nginx/src/letsencrypt

docker compose up -d

---

2. Pedir certificado SSL.

```
docker run -it --rm \
-v /home/ubuntu/docker-volumes/etc/letsencrypt:/etc/letsencrypt \
-v /home/ubuntu/docker-volumes/var/lib/letsencrypt:/var/lib/letsencrypt \
-v /home/ubuntu/docker/letsencrypt-docker-nginx/src/letsencrypt/letsencrypt-site:/data/letsencrypt \
-v "/home/ubuntu/docker-volumes/var/log/letsencrypt:/var/log/letsencrypt" \
certbot/certbot \
certonly --webroot \
--email correo@mail.com --agree-tos --no-eff-email \
--webroot-path=/data/letsencrypt \
-d crespodev.com
```

cd /home/ubunu/docker/letsencrypt-docker-nginx/src/letsencrypt

docker compose down

---

3. Levantar tu web con HTTPS

mkdir -p /home/ubuntu/docker/letsencrypt-docker-nginx/src/production/production-site

vim /home/ubuntu/docker/letsencrypt-docker-nginx/src/production/docker-compose.yml

```
services:
  production-nginx-container:
    container_name: 'production-nginx-container'
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./production.conf:/etc/nginx/conf.d/default.conf
      - ./production-site:/usr/share/nginx/html
      - /home/ubuntu/docker-volumes/etc/letsencrypt/live/crespodev.com/fullchain.pem:/etc/letsencrypt/live/crespodev.com/fullchain.pem
      - /home/ubuntu/docker-volumes/etc/letsencrypt/live/crespodev.com/privkey.pem:/etc/letsencrypt/live/crespodev.com/privkey.pem
    networks:crespodev
      - docker-network

networks:
  docker-network:
    driver: bridge
```

vim /home/docker/docker/letsencrypt-docker-nginx/src/production/production.conf

```
server {
    listen      80;
    listen [::]:80;
    server_name crespodev.com;

    location / {
        rewrite ^ https://$host$request_uri? permanent;
    }

    location ~ /.well-known/acme-challenge {
        allow all;
        root /data/letsencrypt;
    }
}

server {
    server_name crespodev.com;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /etc/letsencrypt/live/crespodev.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/crespodev.com/privkey.pem;

    root /usr/share/nginx/html;
    index index.html;
}
```

cd /home/ubuntu/docker/letsencrypt-docker-nginx/src/production
docker-compose up -d

