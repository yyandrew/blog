---
layout: post
title: "在centos中重置root的密码"
date: 2015-11-14 18:53
comments: true
categories: "system"
---

1. 开机启动时不停按`CTRL+SHIFT+1`直到出现GRUB菜单
2. 在GRUB菜单按`e`
3. 选中第二行，再按`e`
4. 在行尾加上`-s`(注意-s前面有个空格)
5. 按`Enter`再按`b`
6. 进入终端输入`passwd`修改root的密码
