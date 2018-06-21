---
layout: post
title: "ubuntu安装aria2"
date: 2016-12-10 16:21:29 +0800
comments: true
categories: "System"
---

# 通过apt-get安装

```
apt-cache search aria2 # 搜索aria2
sudo apt-get install aria2 # 安装aria2
```

# 通过源码安装

## 安装必需依赖包

``` sh
apt-get install libgnutls-dev nettle-dev libgmp-dev libssh2-1-dev libc-ares-dev libxml2-dev zlib1g-dev libsqlite3-dev pkg-config libcppunit-dev autoconf automake autotools-dev autopoint libtool
```

## 下载源码
```
git clone https://github.com/aria2/aria2.git
```

## 安装
```
autoreconf -i
./configure
make
sudo make install
```
