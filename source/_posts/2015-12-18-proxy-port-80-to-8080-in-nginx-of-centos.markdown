---
layout: post
title: "在centos的nginx中创建一个8080端口到80的映射"
date: 2015-12-18 12:48
comments: true
categories: "nginx"
---
在`/etc/nginx/conf.d/`文件夹新建一个文件`proxy.conf`
内容如下
```
server {
    listen 80;
    server_name your-domain-name.com;
    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass         http://127.0.0.1:8080;
    }
}
```
重启nginx服务器`sudo service nginx restart`
