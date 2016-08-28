---
layout: post
title: Nginx in Docker without Root
---

This post will walk you through how to run Nginx as a non-privileged (i.e., not root) user. We'll use an official Nginx image as a starting point, modify the image using a Dockerfile, and provide some tweaks to the configuration files.

### Overview

When running Nginx as a non-root user, these are the main things to keep in mind:

- User needs read access to website files.
- User needs read and write access to `/var/run/nginx.pid`.
- User needs read and write access to `/var/cache/nginx`.
- Don't listen on ports 80 or 443.

### User and Directory

Before starting, decide which user you will have run the Nginx process and where you want to store the code for the web application. In my example, I'm using the `www-data` user and the directory `/var/www` which is its home directory.

### Dockerfile

```
FROM nginx:stable

COPY ./nginx.conf /etc/nginx/nginx.conf
COPY ./site.conf /etc/nginx/conf.d/default.conf

RUN touch /var/run/nginx.pid && \
  chown -R www-data:www-data /var/run/nginx.pid && \
  chown -R www-data:www-data /var/cache/nginx

USER www-data

VOLUME /var/www
```

This Dockerfile uses the official `nginx:stable` image as its starting point. It then copies in a pair of config files, `/etc/nginx/nginx.conf` and `/etc/Nginx/conf.d/default.conf`. (See below for these files.)

Next, the Dockerfile makes the `www-data` user the owner and group for a few paths that Nginx will need to write to. The first is the PID file `/var/run/nginx.pid`. Since this file doesn't already exist, the Dockerfile uses the `touch` command to create an empty file before setting the ownership. The second path is a directory Nginx uses for various caches.

### Configuration Files

The Dockerfile copies in two files. `/etc/nginx/nginx.conf` is the primary config file for Nginx; `/etc/nginx/conf.d/default.conf` is the default website config file.

You may want to start with the config files provided in the offical image. An easy way to copy the original files from the image to your host is to start a container and use `docker cp`:

```bash
docker run --name nginx -d nginx:stable
docker cp nginx:/etc/nginx/nginx.conf ./nginx.conf
docker cp nginx:/etc/nginx/conf.d/default.conf ./site.conf
docker stop nginx
docker rm nginx
```

Here's a sample `/etc/nginx/nginx.conf` based on the file that comes with the image. Yours may vary a bit, but the only change I needed to make was to remove the `user nginx;` line to avoid a warning since this directive is only meaningfull when Nginx is running as `root`.

```nginx
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

Since the Nginx file for the site will vary widely from site to site, I'm including a minimal version. The important part is `listen` directive.

```nginx
server {
    listen       8080;
    server_name  localhost;
    location / {
        root   /var/www/htdocs;
        index  index.html index.htm;
    }
}
```

Only root can listen on ports below 1024, so you will need to use higher-numbered ports for your site. This is pretty much a non-issue since you'll need to map your host port to your container port anyway, for example:

```bash
docker run -d -p 80:8080 mynginx
```
