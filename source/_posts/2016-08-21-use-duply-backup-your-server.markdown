---
layout: post
title: "使用duply备份文件"
date: 2016-08-21 21:57:06 +0800
comments: true
categories: "System"
---
`duply`是基于`duplicity`文件备份工具，它是利用`Python`实现的。支持增量，对称加密方式来备份文件，支持ftp,ssh,s3,rsync,cifs,webdav,http等协议。
### 安装

#### 安装`duplicity`
``` sh
sudo add-apt-repository ppa:duplicity-team/ppa
sudo apt-get update
sudo apt-get install duplicity
```

#### 安装`duply`
``` sh
wget http://duply.net/tmp/duply.sh -O duply
chmod 755 duply
sudo chown root:root duply
sudo mv duply /usr/bin/
```

### 配置文件目录
```
duply hostname create # 在~/.duply/文件夹下，创建hostname的文件夹
```

`~/.duply/hostname`文件夹目录结构

* conf
* exclude
* post
* pre
* gpg-key.asc

### 创建GPG Key
``` sh
gpg --gen-key
```

``` irc
gpg (GnuPG) 2.0.14; Copyright (C) 2009 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y
......+++
pub   2048R/E3B67367 2016-08-22
      Key fingerprint = F1BD 4CE9 1CD4 02D5 5E34  2AAD C234 81C3 E3B6 7367
uid                  andrew (test.com) <test@test.com>
sub   2048R/599DAFA8 2016-08-22
```

### 备份文件配置

#### 修改`~/.duply/hostname/conf`

为了实现加密备份，你需要上面的`GPG-Key ID`(**E3B67367**)和生成此ID的密码.
``` irc
GPG_KEY='_KEY_ID_'
GPG_PW='_GPG_PASSWORD_'
```
通过设置`GPG_OPTS`可以指定压缩方法和加密方法
``` irc
GPG_OPTS='--compress-algo=bzip2 --personal-cipher-preferences AES256,AES192'
```
设置备份目标地址，`duply`几乎支持所有数据传输协议
``` irc
TARGET='scheme://user[:password]@host[:port]/[/]path'
#   所有协议列表及使用语法
#   file://[/absolute_]path
#   ftp[s]://user[:password]@other.host[:port]/some_dir
#   hsi://user[:password]@other.host/some_dir
#   cf+http://container_name
#   imap[s]://user[:password]@host.com[/from_address_prefix]
#   rsync://user[:password]@other.host[:port]::/module/some_dir
#   # rsync over ssh (only keyauth)
#   rsync://user@other.host[:port]/relative_path
#   rsync://user@other.host[:port]//absolute_path
#   # for the s3 user/password are AWS_ACCESS_KEY_ID/AWS_SECRET_ACCESS_KEY
#   s3://[user:password]@host/bucket_name[/prefix]
#   s3+http://[user:password]@bucket_name[/prefix]
#   # scp and sftp are aliases for the ssh backend
#   ssh://user[:password]@other.host[:port]/some_dir
#   tahoe://alias/directory
#   webdav[s]://user[:password]@other.host/some_dir
```
设置需要备份文件
``` irc
SOURCE='/'
```
其它相关设置
``` irc
MAX_AGE=1Y # 设置过期时效是1年
MAX_FULL_BACKUPS=5 # 最多保持5个备份版本
```

#### 修改备份执行之前脚本及执行之后的脚本

* ~/.duply/hostname/pre

``` sh
  /usr/bin/mysqldump --all-databases -u root -ppw> /tmp/sqldump-$(date '+%F')
```

* ~/.duply/hostname/post

``` sh
  /bin/rm /tmp/sqldump-$(date '+%F')
```

#### 设置白名单及黑名单

* ~/.duply/hostname/exclude

``` sh
+ /etc/ # 备份/etc文件夹
+ /root/ # 备份/root文件夹
+ /var/www/ # 备份/var/www文件夹
- ** # exclude所有其它的文件
```

#### 创建Cronjob

``` sh
0 0 * * 7 /usr/bin/duply /root/.duply/hostname pre_full_verify_purge_post --force # 每星期天做一次完整备份
0 0 * * 1-6 /usr/bin/duply /root/.duply/hostname pre_incr_post # 每星期一至六做一次增量备份
```

#### 小贴士
当你不知道如何使用`duply`时请使用:

``` sh
duply usage
```
