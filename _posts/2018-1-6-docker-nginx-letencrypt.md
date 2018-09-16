---
layout: post
tags: Docker, crypto
title: Docker + Nginx + LetsEncrypt
---
##### Docker + Nginx + LetsEncrypt

Today’s most web applications probably have these characteristics:

- they are complex
- as such, they depend on many moving pieces (e.g. reverse proxy, cache, db, etc)
- require secure connections via TLS

Here are quick tips I found useful to deploy such applications.

### Technologies used:

# -  Docker
# -  Docker Compose
# -  Docker Machine
# -  Nginx
# -  LetsEncrypt (with tool called certbot)

Docker Compose is extremely useful to help manage the complexity of the application’s moving pieces. In case you did not use it before, here is my 2 second pitch. Docker Compose allows to define all of the components in a single configuration therefore allowing for easier maintenance and deployment.
For most use-cases the public-facing component of the application will probably be a reverse proxy. Nginx is one of the most popular reverse proxy servers out there. It is really reliable and lightweight. It often uses <5Mb memory. I’m not sure you can ask for more.

Since stateless applications are cool (12 Factor at all that jazz), nginx should be build as a separate docker compose service. In order words, instead of mounting a volume into the nginx container with the nginx configuration, the container should be built with the configurations baked in. That will make nginx container portable. The resulting Dockerfile will be something like:
```
# ./nginx/Dockerfile

FROM nginx:latest
RUN rm -rf /etc/nginx
COPY ./conf /etc/nginx
EXPOSE 80 443
```
Then it can simply be used within the docker-compose.yml:

```
# ./docker-compose.yml

services:
  nginx:
    build: ./nginx
    container_name: nginx
    ports:
      - 80:80
      - 443:443
  # other compose services
```
Now the application can be deployed anywhere with Docker Compose in combination with Docker Machine:

```
$ eval "$(docker-machine env prod)"
$ docker-compose up -d
```
This is where things get more interesting. In order to configure TLS in nginx container, we need to have certificate installed in the container as well. Copying site’s certificate private key into the container does not seem like a good idea though. The fact that LetsEncrypt issues certificates for only 90 days is great for security but does not help in convenience department. So the question is how to do that without adding too much complexity.

Docker Volumes come to the rescue. Volumes allow to manage data attached to a container independently from the container. That means couple of things:

we can put certificate information on a separate volume which will only be accessible from nginx container
that volume will only have a single copy - in prod - so we will never touch the certificate in dev machines
Adjusted docker-compose.yml will be something like:
```
# ./docker-compose.yml

 services:
   nginx:
     build: ./nginx
     container_name: nginx
+    volumes:
+      - certs:/etc/letsencrypt
+      - certs-data:/data/letsencrypt
     ports:
       - 80:80
       - 443:443
   # other compose services
```
After volumes are configured, we need to adjust nginx settings in order to make it compatible with LetsEncrypt process.
```
# ./nginx/conf/sites-enabled/example.com.conf

server {
    listen      80;
    listen [::]:80;
    server_name example.com;

    location / {
        rewrite ^ https://$host$request_uri? permanent;
    }

    location ^~ /.well-known {
        allow all;
        root  /data/letsencrypt/;
    }
}
```
LetsEncrypt is a free certificate authority which automates the process of creating certs for a site. Its used via a certbot tool. Now that nginx is configured to server ACME challenges from .well-known directory, we can request certificate from LetsEncrypt:
```
$ docker run -it --rm \
      -v certs:/etc/letsencrypt \
      -v certs-data:/data/letsencrypt \
      deliverous/certbot \
      certonly \
      --webroot --webroot-path=/data/letsencrypt \
      -d example.com -d www.example.com
```
Above command will use ACME protocol to issue the certificate for example.com. While doing that it will validate that we are in control of example.com by storing some files in .well-known which nginx will serve. If successfully validated, certificate for example.com will be placed in /etc/letsencrypt which is inside certs volume. Since Nginx will have access to the same certs volume, we can now configure nginx to actually serve the site over TLS:
```
# ./nginx/conf/sites-enabled/example.com.conf

server {
    listen      80;
    ...
}

server {
    listen      443           ssl http2;
    listen [::]:443           ssl http2;
    server_name               example.com www.example.com;

    ssl                       on;

    add_header                Strict-Transport-Security "max-age=31536000" always;

    ssl_session_cache         shared:SSL:20m;
    ssl_session_timeout       10m;

    ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers               "ECDH+AESGCM:ECDH+AES256:ECDH+AES128:!ADH:!AECDH:!MD5;";

    ssl_stapling              on;
    ssl_stapling_verify       on;
    resolver                  8.8.8.8 8.8.4.4;

    ssl_certificate           /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key       /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate   /etc/letsencrypt/live/example.com/chain.pem;

    access_log                /dev/stdout;
    error_log                 /dev/stderr info;

    # other configs
}
```
Above config is btw what I use for my sites which scores A+ in SSLTest.

After a couple of months, the certs can be easily renewed:

```
$ docker run -t --rm \
      -v certs:/etc/letsencrypt \
      -v certs-data:/data/letsencrypt \
      deliverous/certbot \
      renew \
      --webroot --webroot-path=/data/letsencrypt
$ docker-compose kill -s HUP nginx
```
Alternatively cronjob can be used on the host machine to do that automatically every 15 days or so:

```
0 0 */15 * * docker run -t --rm -v certs:/etc/letsencrypt -v certs-data:/data/letsencrypt -v /var/log/letsencrypt:/var/log/letsencrypt deliverous/certbot renew --webroot --webroot-path=/data/letsencrypt && docker kill -s HUP nginx >/dev/null 2>&1
```
Thats it! Its pretty much everything necessary to run site with docker + nginx + LetsEncrypt.

### Side Note:

LetsEncrypt is on a mission to encrypt the whole web so if are able to pitch in financially to them, please consider that.
