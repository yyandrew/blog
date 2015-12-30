---
layout: post
title: "bash rsync command not found的解决方法"
date: 2015-12-30 18:00
comments: true
categories: "System"
---
今天重新的vps的系统之后发现octpress博客不能通过`rake rsync`部署了。提示如下

```sh
bash: rsync: command not found
rsync: connection unexpectedly closed (0 bytes received so far) [receiver]
rsync error: remote command not found (code 127) at /BuildRoot/Library/Caches/com.apple.xbs/Sources/rsync/rsync-47/rsync/io.c(453) [receiver=2.6.9]
```

后来发现是vps重装系统之后忘记安装rysnc了。:(

安装rsync之后问题就解决了。

总结: 要想使用rsync服务<span style="color:red;">两台主机都要安装rsync</span>。
