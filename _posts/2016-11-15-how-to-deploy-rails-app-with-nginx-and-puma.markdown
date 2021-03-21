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

apt-get install curl git build-essential zlibc zlib1g-dev zlib1g libcurl4-openssl-dev libssl-dev libopenssl-ruby libapr1-dev libaprutil1-dev libreadline6 libreadline6-dev -y
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
source /etc/profile.d/rvm.sh  # able to use rvm
rvm install 2.3.1
rvm use 2.3.1
rvm gemset use rate_tool --create
gem install bundler --no-ri --no-rdoc
```

## 安装nginx

``` sh
apt-get install nginx -y
groupadd nginx
usermod -a -G nginx wyb
```

## 安装redis

``` sh
wget http://download.redis.io/releases/redis-stable.tar.gz
tar xzf redis-stable.tar.gz
apt-get install tcl8.5 -y
cd redis-stable
make
make test
make install
cd utils
./install_server.sh

```

## 安装postgresql

``` sh
apt-get install postgresql postgresql-contrib -y
apt-get install libpq-dev -y
su postgres
createuser --createdb --superuser -U postgres wyb

psql -c "ALTER USER wyb WITH PASSWORD ''"

psql -c "create database rate_tool owner wyb encoding 'UTF8' TEMPLATE template0;"

psql -d rate_tool -c "CREATE EXTENSION hstore;"

psql -d rate_tool -c "CREATE EXTENSION pg_trgm;"

RAILS_ENV=production bundle exec rake db:migrate
```

## 安装yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn
```

## 安装monit

``` sh
apt-get install monit -y
```

将下面内容添加到**visudo**文件里，运行`sudo monit summary`命令时不需要输入密码
```
wyb     ALL=NOPASSWD:/usr/bin/monit
```
## import link files
```
scp -r /var/www/rate_tool/shared/config/* wyb@hostname:/var/www/rate_tool/shared/config/
scp -r /etc/monit/conf.d/* wyb@hostname:/etc/monit/conf.d/
scp -r /etc/monit/monitrc wyb@hostname:/etc/monit/monitrc
scp bin/* wyb@hostname:/var/www/rate_tool/shared/bin/
```

## import server scripts
```
mkdir /var/www/rate_tool/shared/scripts
scp wyb@hostname:/var/www/rate_tool/shared/scripts/* /var/www/rate_tool/shared/scripts/
scp /etc/nginx/conf.d/rate_tool.conf root@hostname:/etc/nginx/sites-enabled/
mkdir -p /etc/nginx/ssl/rate_tool
scp /etc/nginx/ssl/rate_tool/* root@hostname:/etc/nginx/ssl/rate_tool/
```

# import images
```
scp -r wyb@hostname:/var/www/rate_tool/shared/public/uploads/* /var/www/rate_tool/shared/public/uploads
```

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

Options:

## Install java on ubuntu

``` sh
apt-get install default-jre
apt-get install default-jdk
```

options:
## Firewall problem on CentOS
```
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
```

Options:
```
# 备份所有的数据库
su - postgres -c 'mkdir /tmp/pg_backup; pg_dumpall > /tmp/pg_backup/dumpall-$(date '+%F').sql'
# 导入所有数据库
psql -f  pgdumpall-2018-11-20.sql postgres
```
