---
layout: post
title: 使用cgdb调试CRuby的C源码
date: 2020-10-30 10:49 +0800
category: Ruby
---

从这篇文章可以学到：

1. 怎么使用cgdb(对GNU gdb进行封装，提供一些加强功能)调试C/C++源码
2. 使用cgdb深入到CRuby源码
3. 从源码编译安装CRuby

### cgdb介绍

调试工具是代码分析中至关重要的工具之一。在使用vim+ctags查看代码时，经常会遇到难以理解的部分，此时，可以借助调试工具，对代码的运行时时行跟踪，通过跟踪运行过程及关键数据的变化，可以从程序执行的过程中理解源代码的功能。

调试C/C++源码的工具有很多种，最常用的是GNU gdb工具。但是个人感觉它不太酷。cgdb对GNU gdb进行了封装，它提供了一些方便地实时查看源代码等其他很有特色的功能，其官网地址为[https://cgdb.github.io/](https://cgdb.github.io/)。它的界面如下图所示：

![screenshot-debugging-with-cgdb.png](/images/screenshot-debugging-with-cgdb.png)

### 编译安装CRuby

一、安装编译依赖

```shell
sudo apt install autoconf gcc automake -y # build-essential make libssl-dev
sudo apt install cgdb -y
```

二、编译并安装CRuby

```shell
wget https://cache.ruby-lang.org/pub/ruby/2.7/ruby-2.7.2.tar.gz
tar xf ruby-2.7.2.tar.gz
cd ruby-2.7.2
autoconf
./configure
make -j8
sudo make install
```

### 使用cgdb调试CRuby

一、先确定要调试哪个ruby API。比如我们想要调试`String#split`这个API。

从[API文档](https://apidock.com/ruby/String/split)来看，它是在`rb_str_split_m`函数里实现了这个功能。

二、使用cgdb调试`rb_str_split_m`函数

* 创建一个tes.rb文件，里面只有一句ruby代码，用来触发`rb_str_split_m`函数。代码如下：

```ruby
# test.rb
puts 'andrew yang'.split
```

* 进入cgdb

```shell
cgdb `which ruby`
```

你会看到下面的这个界面

![start-cgdb.png](/images/start-cgdb.png)

* 在cgdb里查找函数`rb_str_split_m`

```shell
(gdb) info fun rb_str_split_m
All functions matching regular expression "rb_str_split_m":

File string.c:
static VALUE rb_str_split_m(int, VALUE *, VALUE);
```

`info fun`是GNU gdb提供的一个查找函数的命令。其格式为`info fun function_name`

从上面的输出可以看出这个函数在`string.c`被定义的。找到定义的地方之后我们就知道断点应该下在什么地方了。

* 在cgdb里下断点

```shell
(gdb) break string.c:rb_str_split_m
Breakpoint 1 at 0x1904d0: file string.c, line 7896.
```

`break`是GNU gdb提供的下断点的命令。格式比较丰富，这里我们使用`break 文件名:函数名的格式添加了一个断点。更多格式请查看[gdb文档](https://ftp.gnu.org/old-gnu/Manuals/gdb/html_node/gdb_28.html) 

* 在cgdb运行ruby代码(之前创建的test.rb文件)

```shell 
(gdb) run ./test.rb
Starting program: /usr/local/bin/ruby ./test.rb
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, rb_str_split_m (argc=1, argv=0x7ffff7ee7350, str=93824998455240) at string.c:7896
```

从下图可以看出cgdb执行到断点处自动停止了，并且它帮我们自动定位到源码了（红框里的红色的字7896）。

![stop-at-breakpoint-in-cgdb.png](/images/stop-at-breakpoint-in-cgdb.png)

现在你就可以开始查看这个函数运行时的各种状态了。比如我们想查看`result`p这个变量的值，可以使用`display resut`命令

```shell
(gdb) n
(gdb) n
(gdb) display result
1: result = 8
```

### 总结

文章到此就结束了。本文主要使用cgdb来跟踪ruby的运行过程，因此我们事先编译了CRuby源代码，生成可执行程序ruby，利用cgdb对ruby程序的运行进行了跟踪。掌握了cgdb调试C源码的方法之后你就可以开始为ruby修bug了。

### 其它有用的命令：

:point_right:`display`: 查看变量的值。比如要查看变量`x`的值，可以使用`display x`

:point_right:step: 单步跟踪，进入被调用的函数

:point_right:next: 单步跟踪，跳过被调用的函数

:point_right:backtrace: 显示函数调用栈

:point_right:continue: 继续执行到下一个断点。如果没有断点则结束程序。

:point_right:delete: 删除一个断点。格式略多，使用方法可以查看[官方文档](https://ftp.gnu.org/old-gnu/Manuals/gdb/html_node/gdb_31.html)

:point_right:watch: 监视一个表达式。表达式的值发生变化就中断。比如你想监视`x`可以使用`watch x`，之后如果`x`的值发生变化，程序会被中断。

:point_right:ctrl+D：退出cgdb

### 参考链接：

1. [https://sourceware.org/gdb/current/onlinedocs/gdb/](https://sourceware.org/gdb/current/onlinedocs/gdb/)
2. [https://apidock.com/ruby](https://apidock.com/ruby)
3. [https://github.com/ruby/ruby#how-to-compile-and-install](https://github.com/ruby/ruby#how-to-compile-and-install)

