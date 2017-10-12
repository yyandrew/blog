---
layout: post
title: "部署Rails项目(结合nginx和puma)"
date: 2016-11-15 11:09:40 +0800
comments: true
categories: "Rails"
---

# 准备好服务器上的部署账号及目录结构

## 安装必要的包

``` sh
apt-get update
apt-get install curl git build-essential zlibc zlib1g-dev zlib1g libcurl4-openssl-dev libssl-dev libopenssl-ruby libapr1-dev libaprutil1-dev libreadline6 libreadline6-dev
```

## 创建新的用户

``` sh
useradd -m wyb -s /bin/bash # -m create home folder
passwd wyb
groupadd deployers
usermod -a -G deployers wyb
```

## 更改默认编辑器

``` sh
update-alternatives --config editor
```

## 将用户**wyb**添加到**sudoer**

将下面的内容

```
wyb     ALL=(ALL:ALL) ALL
```

添加到**visudo**

## 配置ssh无密码登录

``` sh
cat ~/.ssh/id_rsa.pub | ssh wyb@hostname "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

## 创建部署目录

``` sh
mkdir /var/www
chown -R wyb:deployers /var/www
chmod -R g+w /var/www
```

## 安装rvm

```sh
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer | bash -s stable
rvm install 2.3.1
rvm use 2.3.1
rvm gemset use gh60-club --create
gem install bundler --no-ri --no-rdoc
source /etc/profile.d/rvm.sh  # able to use rvm

```

## 安装nginx

``` sh
apt-get install nginx
groupadd nginx
usermod -a -G nginx wyb
```

## 安装redis

``` sh
wget http://download.redis.io/releases/redis-stable.tar.gz
tar xzf redis-stable.tar.gz
apt-get install tcl8.5
cd redis-stable
make
make test
make install
cd utils
./install_server.sh

```

## 安装postgresql

``` sh
apt-get install postgresql postgresql-contrib
apt-get install libpq-dev
su postgres
createuser --createdb --superuser -Upostgres wyb

psql -c "ALTER USER wyb WITH PASSWORD ''"

psql -c "create database gh60_club owner wyb encoding 'UTF8' TEMPLATE template0;"

psql -d gh60_club -c "CREATE EXTENSION hstore;"

psql -d gh60_club -c "CREATE EXTENSION pg_trgm;"

RAILS_ENV=production bundle exec rake db:migrate
```

## 安装monit

``` sh
apt-get install monit
```

## import link files
scp wyb@hostname:/var/www/gh60-club/shared/config/* /var/www/gh60-club/shared/config/

## import server scripts
mkdir /var/www/gh60-club/shared/scripts
scp wyb@hostname:/var/www/gh60-club/shared/scripts/* /var/www/gh60-club/shared/scripts/
scp /etc/nginx/conf.d/gh60-club.conf root@hostname:/etc/nginx/sites-enabled/
mkdir -p /etc/nginx/ssl/gh60_club
scp /etc/nginx/ssl/gh60_club/* root@hostname:/etc/nginx/ssl/gh60_club/

# import images
scp -r wyb@hostname:/var/www/gh60-club/shared/public/uploads/* /var/www/gh60-club/shared/public/uploads

# puma常用命令

``` sh
bundle exec pumactl -F config/puma.rb start
bundle exec pumactl -F config/puma.rb status
bundle exec pumactl -F config/puma.rb stop
```
# tips

## 1. There is no public key available for the following key IDs:

[解决方法](http://unix.stackexchange.com/a/209266)

``` sh
apt-get install debian-archive-keyring
apt-key update
```

## 2.Upgrade debian to the latest version in one line

``` sh
apt-get update;apt-get upgrade;wget -q https://www.rootusers.com/wp-content/uploads/2015/08/update.txt -O /etc/apt/sources.list;apt-get update;apt-get upgrade;apt-get dist-upgrade;apt-get autoremove;cat /etc/debian_version;echo "The above number shows the current Debian version. It is highly recommended that you reboot the system."
```
