---
layout: post
title: "让浏览器强制请求最新的资源文件而不是使用缓存"
date: 2017-08-03 15:11:44 +0800
comments: true
categories: "Rails"
---

### 故障场景
用户上传更新头像之后，头像图片依旧使用旧图片。当用户在新tab打开页面故障消失。
### 原因
打开Chrome开发者工具，在Network面板中查看图片请求，发现请求“Header"->"General"->"Status Code""的值是**200  (from memory cache)**
### 解决方法
1, IE Firefox(Chrome Safari无效)可以给reload方法添加true参数强制从服务器请求资源。如`window.location.reload(true)`

2, 通用方法：在请求地址添加一个时间戳参数。如http://foo.bar/assets/avata.png?upd=123456 (123456最好用时间戳代替)
