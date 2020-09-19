---
layout: post
title: "Rails中session的三种存储方式"
date: 2016-12-13 17:01:23 +0800
comments: true
categories: "Rails"
---

众所周知，Rails中session可以以三种方式存储：cookie，cache，database。我们来说说这三种方式的使用场景和优缺点.

### 存储在cookies中

这种方式无疑是最简单的，无需要任何配置。大多数情况下是够用的。

### 存储在cache中

如果你的项目已经使用的`Memcache`，那恭喜你，这个方法也不难使用，因为环境已经搭好了。

*优点*
1. 不需要人为控制，也就是说你不用写多余的代码去清除这些session，因为`Memcache`会自动帮你清除旧数据。
2. 效率高，因为是存储在内存里

*缺点*

1. 如果旧数据对你来说有价值，这种方法就不合适了，因为旧的数据会被删除
2. 内存必要要足够大


### 存储在数据库中

如果你想保留session直到你想使它失效，这时就可以考虑把session存储在数据库中了。比如redis，ActiveRecord等

*优点*

1. 有完全的控制权

*缺点*

1. 你需要手动(写代码)去删除这些session
2. 你也许会存储大量无用的session(比如googlebot会产生大量的session)
