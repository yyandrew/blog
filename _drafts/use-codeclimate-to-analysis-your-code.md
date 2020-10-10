---
layout: post
title: Use codeclimate to analysis your code
---
依赖

1. Docker

## 安装

```shell 
curl -L https://github.com/codeclimate/codeclimate/archive/master.tar.gz | tar xvz
cd codeclimate-* && sudo make install
```

## 配置

```shell
cp .codeclimate.yml /your-app-root-folder/.codeclimate.yml
```

## 开始分析

```shell 
cd /your-app-root-folder
codeclimate engines:install
codeclimate analyze
```

