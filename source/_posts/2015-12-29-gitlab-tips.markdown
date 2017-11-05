---
layout: post
title: "gitlab tips"
date: 2015-12-29 16:22
comments: true
categories: "Git"
---
* 利用[Omnibus package installation](https://about.gitlab.com/downloads/#ubuntu1404)只需2分钟可以安装完成
* gitlab配置文件默认在`/etc/gitlab/gitlab.rb`和`/var/opt/gitlab/gitlab-rails/etc/gitlab.yml`
* 进入production数据库`sudo -u gitlab-psql psql -h /var/opt/gitlab/postgresql -d gitlabhq_production`
* 手动激活一个新用户`gitlabhq_production=#  update users set confirmed_at=NOW() where email='example@example.com';`
* 禁止gitlab开机自动启动`systemctl disable gitlab-runsvdir.service`
