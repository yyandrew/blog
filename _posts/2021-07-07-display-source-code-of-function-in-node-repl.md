---
layout: post
title: 查看 node 源代码只需要两步
date: 2021-07-07 14:24 +0800
---
比如想要看看 `http.createServer` 是如何实现的，可以通过下面简单两步实现。

1. 在命令行直接输入 `node` 进入 node 的交互式的编程环境(REPL)
2. 在 REPL 中输入下面代码查看 createServer 的源码
``` js
let http = require('http')
console.log(http.createServer.toString())
```

输出示例：

![show-source-code-of-nodejs.png](/images/show-source-code-of-nodejs.png)
