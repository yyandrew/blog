---
layout: post
title: "not authorized to control networking"
date: 2015-08-30 15:59
comments: true
categories: "System"
---
Network-manager是Debian上非常优秀的网络管理工具。

然而Debian中默认只有root用户可以管理，如果就非root用户企图修改就会出现错误。如下图：

![error.png](/images/error.png)

解决方法如下：

1. 将用户添加到netdev组`addgroup user_name netdev`，不要忘记替换***user_name***
2. 修改文件`/etc/polkit-1/localauthority/50-local.d/org.freedesktop.NetworkManager.pkla`，如果没有这个文件就新建
        # /etc/polkit-1/localauthority/50-local.d/org.freedesktop.NetworkManager.pkla
        [nm-applet]
        Identity=unix-group:netdev
        Action=org.freedesktop.NetworkManager.*
        ResultAny=yes
        ResultInactive=no
        ResultActive=yes
