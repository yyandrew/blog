---
layout: post
title: "在ubuntu安装Zabbix server并配置agent监控多VPS"
date: 2016-12-04 12:09:09 +0800
comments: true
categories: "Server"
---
## 简介

`Zabbix`是一个监控应用，可以用来实时监控你的服务器和VPS主机。

我们假设`Zabbix`服务器的IP地址用*192.168.0.1*，被监控的服务器地址用*192.168.1.1*

## 安装必要依赖包

``` sh
apt-get install build-essential libxml2-dev
```

## 安装Zabbix服务器

Zabbix服务器相当于一个监控站点，可以查看所有的服务器或者VPS主机的系统信息。

修改系统的source list增加PPA

``` sh
sudo vi /etc/apt/sources.list
```

将下面的内容增加到source.list的文件末尾

```
# Zabbix Application PPA
deb http://ppa.launchpad.net/tbfr/zabbix/ubuntu precise main
deb-src http://ppa.launchpad.net/tbfr/zabbix/ubuntu precise main
```

增加PPA的key

``` sh
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C407E17D5F76A32B
```

安装`Zabbix`，由于网站是用php来实现的所以你需要安装LAMP

``` sh
sudo apt-get update
sudo apt-get install zabbix-server-mysql php5-mysql zabbix-frontend-php
```

## 配置Zabbix服务器

修改`Zabbix`服务器的配置

``` sh
sudo vi /etc/zabbix/zabbix_server.conf
```

搜索下面的配置内容，并修改

```
DBName=zabbix
DBUser=zabbix
DBPassword=我是密码
```

## 配置MySQL

进入程序的目录，解压SQL文件

``` sh
cd /usr/share/zabbix-server-mysql/
sudo gunzip *.gz
```

导入SQL

```
mysql -u root -p
```

为`Zabbix`创建新用户, 对照`/etc/zabbix/zabbix_server.conf`文件的设置

``` sh
create user 'zabbix'@'localhost' identified by '我是密码';
```

创建数据库

``` sh
create database zabbix;
```

设置数据库权限

``` sh
grant all privileges on zabbix.* to 'zabbix'@'localhost';
```

使修改生效

``` sh
flush privileges;
```

退出MySQL

``` sh
exit;
```


继续导入`Zabbix`初始数据

``` sh
mysql -u zabbix -p zabbix < schema.sql
mysql -u zabbix -p zabbix < images.sql
mysql -u zabbix -p zabbix < data.sql
```


## 配置PHP

修改`/etc/php5/apache2/php.ini`中的下面属性

``` sh
post_max_size = 16M
max_execution_time = 300
max_input_time = 300
date.timezone = UTC
```

复制`Zabbix`提供的php配置文件

``` sh
sudo cp /usr/share/doc/zabbix-frontend-php/examples/zabbix.conf.php.example /etc/zabbix/zabbix.conf.php
```

修改`/etc/zabbix/zabbix.conf.php`

``` sh
$DB['DATABASE'] = 'zabbix';
$DB['USER'] = 'zabbix';
$DB['PASSWORD'] = '我是密码'
```

## 配置其它文件

`Zabbix`的apache配置文件

``` sh
sudo cp /usr/share/doc/zabbix-frontend-php/examples/apache.conf /etc/apache2/conf.d/zabbix.conf
```

启动Apache的"alias"模块

``` sh
sudo a2enmod alias
```

重启`Apache`服务

``` sh
sudo service apache2 restart
```

修改`Zabbix`

``` sh
sudo vi /etc/default/zabbix-server
```

修改"START"的值为"yes"

``` sh
START=yes
```

重启`Zabbix`

``` sh
sudo service zabbix-server start
```

## 安装并配置Zabbix Agent

`Zabbix Agent`顾名思义是用用收集被监控的服务器或者VPS主机的系统信息并发给`Zabbix`服务器用于查看

安装agent应用

``` sh
sudo apt-get update
sudo apt-get install zabbix-agent
```

修改agent配置文件

``` sh
sudo vi /etc/zabbix/zabbix_agentd.conf
```

修改"Server"属性

``` sh
Server=192.168.0.1
```

修改"Hostname", 它对应被监控的服务器或者VPS主机

``` sh
Hostname=192.168.1.1
```

重启`Zabbix`agent

``` sh
sudo service zabbix-agent restart
```


## 登录到web

默认的地址为: [192.160.0.1/zabbix](http://192.160.0.1/zabbix)

默认admin用户

``` sh
Username = admin
Password = zabbix
```
