---
layout: post
title: "如何使用git svn"
date: 2016-05-16 15:43:57 +0800
comments: true
categories: "Git"
---

最近在做一个朋友介绍的项目时候得知项目用的版本控制是svn。

``` bash
git svn clone http://yourrepository/ -T trunk -b branches -t tags # 克隆远程库代码到本地
git commit -am 'Adding git-svn instructions to the README' # 提交修改
git svn dcommit #推送修改到远程库
```
