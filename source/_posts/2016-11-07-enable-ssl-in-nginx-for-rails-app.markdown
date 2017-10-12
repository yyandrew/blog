---
layout: post
title: "给rails应用添加https访问"
date: 2016-11-07 14:40:32 +0800
comments: true
categories: "Rails"
---
Rails的服务器选用的是`thin`
``` irc
upstream app_name_thin_cluster {
  server unix:/var/www/app_name/shared/tmp/sockets/thin.0.sock;
  server unix:/var/www/app_name/shared/tmp/sockets/thin.1.sock;
  server unix:/var/www/app_name/shared/tmp/sockets/thin.2.sock;
  server unix:/var/www/app_name/shared/tmp/sockets/thin.3.sock;
  server unix:/var/www/app_name/shared/tmp/sockets/thin.4.sock;
  server unix:/var/www/app_name/shared/tmp/sockets/thin.5.sock;
}
server {
  listen   80;
  server_name  app.com;
  # force http request redirect to https
  return 301 https://$server_name$request_uri;
  access_log /var/www/app_name/shared/log/nginx_access.log;
  error_log /var/www/app_name/shared/log/nginx_error.log;

  root /var/www/app_name/current/public;
  index  index.html;

  location ~* \.(eot|ttf|woff)$ {
      add_header Access-Control-Allow-Origin *;
  }

  # Add expires header for static content
  location ~* \.(js|css|jpg|jpeg|gif|png|swf|woff2)$ {
     if (-f $request_filename) {
      expires      max;
      break;
    }

    if (!-f $request_filename) {
      proxy_pass http://app_name_thin_cluster;
      break;
    }
  }

  # this rewrites all the requests to the maintenance.html
  # page if it exists in the doc root. This is for capistrano's
  # disable web task
  if (-f $document_root/system/maintenance.html) {
    rewrite  ^(.*)$  /system/maintenance.html last;
    break;
  }

  location / {
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    if (-f $request_filename/index.html) {
      rewrite (.*) $1/index.html break;
    }

    if (-f $request_filename.html) {
      rewrite (.*) $1.html break;
    }

    if (!-f $request_filename) {
      proxy_pass http://app_name_thin_cluster;
      break;
    }
  }
}
server {
  listen 443;
  server_name app.com;
  ssl on;
  ssl_certificate /etc/nginx/ssl/app_name/ssl-bundle.crt;
  ssl_certificate_key /etc/nginx/ssl/app_name/app_name.key;
  ssl_prefer_server_ciphers on;
  root /var/www/app_name/current/public;
  proxy_redirect off;
  location / {
    try_files $uri/index.html $uri.html $uri @cluster;
  }
  location @cluster {
    proxy_pass http://app_name_thin_cluster;
  }
}
```
