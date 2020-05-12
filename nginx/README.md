# Nginx Configuration

Below are the steps taken to achieve the nginx configuration for the server.

## Running Simple http Server

TODO

## Letsencrypt Certbot Setup

Below are the steps taken to setup the letsencrypt certbot for automated SSL certificate enforecement. They assume you are starting with a very basic configuration (nginx, and a single submodule, say matthewvarga.net). An example of the starting docker-compose.yaml file looks like:

```
version: "3.6"

services:
  matthewvarga:
    build: "./matthewvarga.net/."
    ports:
      - "8080"

  nginx:
    build: "./nginx"
    ports:
      - "80:80"
    depends_on:
      - "matthewvarga"

```

and nging.conf:

```
events {
    worker_connections 1024;
}
http {
  server_tokens off;
  server {
    listen 80;
    listen [::]:80;

    root  /var/www;
    server_name matthewvarga.net www.matthewvarga.net;

    location / {
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header Host            $http_host;
      proxy_pass http://matthewvarga:8080/;
    }
  }
}
```

## Steps to add and enable letsencypt certbot docker container

1) Add the following to http server in the nginx.conf file:

    ```
    location ~ /.well-known/acme-challenge {
      allow all;
      root /var/www;
    }
    ```

2) Add the following volumes to the nginx container in docker-compose.yaml:

    ```
    volumes:
        - certbot-etc:/etc/letsencrypt
        - certbot-var:/var/lib/letsencrypt
        - webroot:/var/www
    ```

3) Add the following container to docker-compose.yaml for letsencrypt certbot:

    ```
    certbot:
    image: certbot/certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
      - webroot:/var/www
    depends_on:
      - nginx
    command: certonly --webroot --webroot-path=/var/www --email matthew.varga@mail.utoronto.ca --agree-tos --no-eff-email --staging -d matthewvarga.net -d www.matthewvarga.net
    ```

    **IMPORTANT:** make note of the --staging flag in the command line

4) Run `docker-compose up -d` to verify our domain requests are successful. Next run `docker-compose ps`. If everything is successul, there should be three containers listed (nginx with state Up, matthewvarga with state Up, and certbot with state Exit).

5) Run `docker exec <nginx_container_id> ls -la /etc/letsencrypt/live` where the container Id can be found by running `docker ps`. The output should contain the following 4 items:
- .
- ..
- README
- matthewvarga.net

6) Now that we now the requests will be successul, replace `--staging` with `--force-renewal` in the certbot command in docker-compose.yaml. 

7) Run `docker-compose up --force-recreate --no-deps certbot` to recreate the certbot container.

8) Now stop the nging container with `docker-compose stop nginx`, and create the directory for our Diffie-Helman key: `mkdir dhparam`.

9) Run the following command to gnerate Diffie-Helman key output into the directory we just created:

    `sudo openssl dhparam -out <PROJECT_ROOT>/dhparam/dhparam-2048.pem 2048`\
    where <PROJECT_ROOT> is the location you have cloned the project.

10) update nginx.conf to redirect from http to https, and add additional headers for security measures.
The old file should look like:
    ```
    events {
        worker_connections 1024;
    }
    http {
    server_tokens off;
    server {
        listen 80;
        listen [::]:80;

        root  /var/www;

        server_name matthewvarga.net www.matthewvarga.net;

        location / {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host            $http_host;
        proxy_pass http://matthewvarga:8080/;
        }

        location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www;
        }
    }
    }
    ```
    The new file should look like:
    ```
    events {
        worker_connections 1024;
    }

    # http configuration
    server {
    listen 80;
    listen [::]:80;
    
    server_name matthewvarga.net www.matthewvarga.net matthew-varga.com www.matthew-varga.com;

    # letsencrypt ssl challenge file locations
    location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www;
    }

    # redirect all other traffic to https
    location / {
        rewrite ^ https://$host$request_uri? permanent;
    }
    }

    # https configuration
    server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    server_name matthewvarga.net www.matthewvarga.net matthew-varga.com www.matthew-varga.com;

    # ssl cert locations
    ssl_certificate /etc/letsencrypt/live/matthewvarga.net/fullchain.pem
    ssl_certificate_key /etc/letsencrypt/live/matthewvarga.net/privkey.pem

    ssl_certificate /etc/letsencrypt/live/matthew-varga.com/fullchain.pem
    ssl_certificate_key /etc/letsencrypt/live/matthew-varga.com/privkey.pem

    ssl_buffer_size 8k;

    # Diffie-Helman credentials and key location
    ssl_dhparam /etc/ssl/certs/dhparam-2048.pem;

    ssl_protocols TLSv1.2 TLSv1.1 TLSv1;
    ssl_prefer_server_ciphers on;

    ssl_ciphers ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    ssl_ecdh_curve secp384r1;
    ssl_session_tickets off;

    ssl_stapling on;
    ssl_stapling_cerify on;
    resolver 8.8.8.8;

    # point requests to aliased matthewvarga container
    location / {
        try_files $uri @matthewvarga;
    }

    # server matthewvara container, and add additional security headers
    location @matthewvarga {
        proxy_pass http://matthewvarga:8080;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy "default-src * data: 'unsafe-eval' 'unsafe-inline'" always;
        # add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
        # enable strict transport security only if you understand the implications
    }

    root /var/www;
    }

    ```


**NOTE**: If there is an issue, check the docker logs with `docker-compose logs <service_name>` where the service_name is optional.


https://www.digitalocean.com/community/tutorials/how-to-secure-a-containerized-node-js-application-with-nginx-let-s-encrypt-and-docker-compose