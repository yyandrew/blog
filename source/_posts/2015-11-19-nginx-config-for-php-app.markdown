---
layout: post
title: "运行php的nginx配置文件"
date: 2015-11-19 05:54
comments: true
categories: "php"
---
nginx运行php应用的配置的示例

首先备份nginx默认的配置`cp /etc/nginx/sites-enabled/default /etc/nginx/sites-enabled/default.bak`

然后替换`/etc/nginx/sites-enabled/default`文件的内容

```bash
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html; # 你的网站放在这个文件夹下
    index index.php index.html;
    location / {
        try_files $uri $uri/ = 404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /var/www/html;
    }
    location ~ \.php$ {
        try_files $uri = 404;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}


```
