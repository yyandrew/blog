---
layout: post
title: "django cheatsheet"
date: 2016-08-07 15:02:26 +0800
comments: true
categories: "Python"
---
### 查看是不是已经安装`django`
```sh
python -m django --version
```
### 安装`django`
```sh
python install django
```
### 新建一个`django`项目
```sh
django-admin startproject APP_NAME
```
### 启动项目
```sh
python manage.py runserver
```
### 访问项目 [129.0.0.1:8000](http://127.0.0.1:8000)
### 新建一个新的`app`
```sh
python manage.py startapp main_app # 新建一个名称为`main_app`的应用
```
### migration
```sh
python manage.py makemigration # 创建一个新的migration文件
python manage.py migrate # 运行migration
```
