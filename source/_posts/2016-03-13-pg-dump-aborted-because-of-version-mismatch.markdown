---
layout: post
title: "由于pg_dump版本不一致导致pg_dump命令中断问题 "
date: 2016-03-13 20:55
comments: true
categories: "Rails"
---
最近在导出postgresql数据库的时候遇到一个问题，
大致的错误信息如下：
```sh
pg_dump --no-acl --no-owner  -U user_name  app_name | bzip2 - - > db/app_name_2016-03-13-204024.sql.bz2
pg_dump: server version: 9.4.6; pg_dump version: 8.4.20
pg_dump: aborting because of server version mismatch
```
仔细看这条错误信息发现这个问题其实是由于pg_dump版本不一致导致的
知道了原因所以解决起来也不会太复杂，只要为系统自带的`/usr/bin/pg_dump`添加一个超链接就可以解决问题了

1.查找系统上已经安装的所有pg_dump版本
```sh
sudo find / -type f -name pg_dump
```
我的系统的输出信息是这样的
```sh
/usr/bin/pg_dump
/usr/pgsql-9.4/bin/pg_dump
```
2.上面的输出信息表明已经有两个版本的`pg_dump`了, 我们只用运行下面命令为`/usr/bin/pg_dump`创建一个链接就可以了
```sh
sudo ln -s /usr/pgsql-9.4/bin/pg_dump /usr/bin/pg_dump --force
```
