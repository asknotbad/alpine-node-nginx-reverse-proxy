# alpine-node-nginx-reverse-proxy
Docker Alpine container with nginx reverse-proxy

## docker-compose.yml

```yaml
  version: '3.8'

  services:
    reverseproxy:
      container_name: nginxreverseproxy
      restart: always
      build:
        context: ./docker
      volumes:
        # if you need to change default nginx.conf
        - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
        # if you need to replace default conf.d folder contents
        - ./docker/nginx/conf.d:/etc/nginx/conf.d
        - ./docker/nginx/sites-available:/etc/nginx/sites-available
        # your sites configs
        - ./docker/nginx/sites-enabled:/etc/nginx/sites-enabled
        # log files
        - ./docker/log:/var/log/nginx
      ports:
        - 80:80
        - 443:443
        # any other ports..
```

## default configs

<code>docker/nginx.conf</code>:

```nginx
user  nginx;
worker_processes  auto;
include /etc/nginx/modules-enabled/*.conf;

error_log  /var/log/nginx/error.log notice;
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
    keepalive_timeout  65;
    gzip  on;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

```

<code>docker/conf.d/default.conf</code>:

```nginx
  server {
    listen 80 default_server;
    # listen [::]:80 default_server;
    # listen 443 ssl;
    server_name _;
    gzip on;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_proxied any;
    gzip_vary on;
    gzip_types
      application/atom+xml
      application/javascript
      application/json
      application/ld+json
      application/manifest+json
      application/rss+xml
      application/vnd.geo+json
      application/vnd.ms-fontobject
      application/x-font-ttf
      application/x-javascript
      application/x-web-app-manifest+json
      application/xhtml+xml
      application/xml
      font/opentype
      image/bmp
      image/svg+xml
      image/x-icon
      text/cache-manifest
      text/css
      text/javascript
      text/plain
      text/vcard
      text/vnd.rim.location.xloc
      text/vtt
      text/x-component
      text/x-cross-domain-policy;
      # text/html is always compressed by gzip module

    # text/html is always compressed by gzip module
    location ~*  \.(jpg|jpeg|png|gif|ico|css|js|pdf)$ {
      expires 7d;
    }

    root /var/www/html;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html index.php;

    location / {
      # First attempt to serve request as file, then
      # as directory, then fall back to displaying a 404.
      try_files $uri $uri/ =404;
      #return 301 https://$host$request_uri;
    }
  }

```

## sample config for reverse-proxy

<code>docker/sites-enabled/reverse-proxy.conf</code> (can be renamed):

```nginx
  server {
    listen 80;
    ## required hostname
    server_name 0.0.0.0 localhost 127.0.0.1;

    access_log /var/log/nginx/reverse-access.log;
    error_log /var/log/nginx/reverse-error.log;

    location / {
      proxy_pass https://asknotbad.com;
      ## headers (if necessary)
      # proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

    }
  }
```
