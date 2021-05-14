---
layout: post
title: 记一次重命名一个已经上线的Rails项目
date: 2021-05-14 11:58 +0800
---

最近有个项目来了一个需求，客户觉得项目的仓库名称和项目需求文档里的名字不一致，导致客户理解困难，所以他们希望能够使代码仓库，网站域名及需求文档里的项目名称能够保持一致。

这次修改分为以下几个步骤：

1. 查找并且替换代码库里的旧项目名；
2. 在服务器上创建新的数据库；
3. 修改 Capistrano 自动部署配置文件；
4. 在服务器上添加新的 monit 配置文件；
5. 在服务器上添加新的 nginx 配置文件；
6. 添加新的 DNS 记录，并配置 SSL 证书启动 https 安全协议；
7. 在 gitlab 修改仓库的名称。

#### 查找并且替换代码库里的旧项目名

不用多说，查找替换是任何编辑器都有的基本功能。

#### 在服务器上创建新的数据库

由于我们功能使用的PostgreSQL，这们可以使用`CREATE DATABASE target_database
WITH TEMPLATE source_database;`这条 SQL 命令将根据旧的数据库来创建新的数据库。

#### 修改 Capistrano 自动部署配置文件

因为要创建新的 gemset 所以需要修改 `rvm_ruby_version`。

修改`server`为最新的 domain 。

如果是测试阶段还需要修改 `branch` ，它的值应该是新创建的分支。

#### 在服务器上添加新的 monit 配置文件

monit使用了 systemd 的来启动/停止 rails server ，所以要先创建一个 systemd 的服务。在创建一个新的文件`/etc/systemd/system/new_app_puma.service`内容如下：

```shell
[Unit]
Description=Puma HTTP Server
After=network.target

[Service]
Type=simple

# If your Puma process locks up, systemd's watchdog will restart it within seconds.
# WatchdogSec=10

# Preferably configure a non-privileged user
User=www-data
Group=www-data
UMask=0002

WorkingDirectory=/var/www/develop/new_app/current

# Helpful for debugging socket activation, etc.
Environment=DEPLOY_TO=/var/www/develop/new_app
Environment=RVM=2.7.1@new_app_develop

# SystemD will not run puma even if it is in your path. You must specify
# an absolute URL to puma. For example /usr/local/bin/puma
# Alternatively, create a binstub with `bundle binstubs puma --path ./sbin` in the WorkingDirectory
ExecStart=/usr/local/rvm/bin/rvm $RVM do bundle \
          exec puma -C config/puma.rb -e develop

Restart=always

[Install]
WantedBy=multi-user.target
```

运行 `sudo systemctl daemon-reload`重新载入配置文件，使修改生效。

运行`sudo systemctl enable new_app_puma.service`将 service 设置为随系统启动而自动重启。

创建新的 monit 配置文件。

```shell
check process new_app_develop_puma with pidfile /var/www/develop/new_app/shared/pids/puma.pid
    start program = "/bin/systemctl start new_app_develop_puma"
    stop program  = "/bin/systemctl stop new_app_develop_puma"
    if cpu > 60% for 2 cycles then alert
    if cpu > 80% for 5 cycles then restart
    if totalmem > 0.5 GB for 5 cycles then restart
    if 3 restarts within 5 cycles then timeout
    group new_app_develop_ruby
    group new_app_develop_puma
    group new_app_develop
    group all_develop_ruby
```

运行`sudo systemctl restart monit`重启 monit ，使用配置文件生效。

运行`sudo monit reload` 重新加载monitrc文件，重启deamon进程。

#### 在服务器上添加新的 nginx 配置文件

新建文件 `/etc/nginx/sites-enabled/develop/new_app.conf `，内容如下：

```
upstream new_app_develop_puma_cluster {
  server  unix:/var/www/develop/new_app/shared/sockets/puma.socket;
}

server {
  listen   80;
  listen   [::]:80;
  server_name  new_app.com *.new_app.com;

  access_log /var/www/develop/new_app/shared/log/nginx_access.log;
  error_log /var/www/develop/new_app/shared/log/nginx_error.log;

  root /var/www/develop/new_app/current/public;
  index  index.html;

  location /robots.txt { alias /etc/nginx/robots.txt; }

  location ~ /.well‐known/acme‐challenge {
    root /var/www/develop/new_app/shared/public;
    allow all;
  }

  location ~ (.php|.aspx|.asp|myadmin) {
      return 404;
  }

  location ~* .(js|css|jpg|jpeg|gif|png|swf)$ {
     if (‐f $request_filename) {
      expires      max;
      break;
    }

    if (!‐f $request_filename) {
      proxy_pass http://new_app_develop_puma_cluster;
      break;
    }
  }

  if (‐f $document_root/system/maintenance.html) {
    rewrite  ^(.*)$  /system/maintenance.html last;
    break;
  }

  location ~* .(eot|ttf|woff)$ {
      add_header Access‐Control‐Allow‐Origin *;
  }

  location / {
    rewrite ^(.*) https://new_app.com$1 permanent;
  } }

server {
  listen   443 ssl http2;
  listen   [::]:443 ssl http2;
  server_name  new_app.com;
  ssl_certificate        /etc/letsencrypt/live/new_app.com/fullchain.pem;
  ssl_certificate_key    /etc/letsencrypt/live/new_app.com/privkey.pem;
  ssl_session_timeout  5m;
  ssl_protocols TLSv1.2 TLSv1.1;
  ssl_ciphers ’EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH’;
  ssl_session_cache shared:TLS:2m;
  ssl_stapling on;
  ssl_stapling_verify on;
  resolver 8.8.8.8;
  add_header  Strict‐Transport‐Security  ’max‐age=31536000;   includeSubDomains’;
  ssl_dhparam /etc/nginx/ssl/dhparam.pem;
  ssl_prefer_server_ciphers   on;
  access_log      /var/www/develop/new_app/shared/log/nginx_ssl_access.log;
  error_log       /var/www/develop/new_app/shared/log/nginx_ssl_error.log;

  root /var/www/develop/new_app/current/public;
  index  index.html;

  location /robots.txt { alias /etc/nginx/robots.txt; }

  location ~ ^/assets/  {
    expires      max;
    try_files $uri http://new_app_develop_puma_cluster;
  }

  if (‐f $document_root/system/maintenance.html) {
    rewrite  ^(.*)$  /system/maintenance.html last;
    break;
  }

  location /plugin_assets {
    if (‐f $request_filename) {
      expires      max;
      break;
    }
  }

  location / {
    proxy_set_header  X‐Real‐IP  $remote_addr;
    proxy_set_header  X‐Forwarded‐For $proxy_add_x_forwarded_for;
    proxy_set_header X‐Forwarded‐Proto $scheme;

    proxy_set_header Host $http_host;
    proxy_redirect off;

    if (‐f $request_filename/index.html) {
      rewrite (.*) $1/index.html break;
    }

    if (‐f $request_filename.html) {
      rewrite (.*) $1.html break;
    }

    if (!‐f $request_filename) {
      proxy_pass http://new_app_develop_puma_cluster;
      break;
    }
  } }
```

#### 添加新的 DNS 记录，并配置 SSL 证书启动 https 安全协议

证书可以使用`certbot`自动申请 [Let’s Encrypt](https://letsencrypt.org/) 的免费证书。

#### 在 gitlab 修改仓库的名称

打开 gitlab 的 setting 页面 ，在这里可以重新命名项目名。
