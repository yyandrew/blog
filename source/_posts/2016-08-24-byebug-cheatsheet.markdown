---
layout: post
title: "byebug小抄"
date: 2016-08-24 20:42:39 +0800
comments: true
categories: "Rails"
---

### 添加断点
``` ruby
... code ...
debugger
# or
# byebug
... code ...
```
### 重复上一次命令
按`Enter`

### 退出当前程序
`q` 或者 `kill`

### 其它常用命令

* `c <linenumber>` 继续运行直到代码结束或者到指定的**linenumber**
* `n <linenumber>` 继续运行到下一行或者指定的**linenumber**
* `s <number>` 进入下一行或者进入下**number**步
* `b` 显示stack trace
* `l <option>` 显示当前断点之后的代码, option包含`-`:显示断点之前的代码， `=`:显示当前断点附近的代码, `<first>-<last>`:显示first到last之间的代码
* `th <option>` 显示当前进程信息, option包含`l`:列出所有进程，`stop <number>`:终止进程**number**
* `m class|module` 显示class或者module的实例方法
* `m i object` 显示对象的方法
* `h <command-name>` 显示帮肋信息
