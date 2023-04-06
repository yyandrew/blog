---
layout: post
title: 'Mysql2::Error: Specified key was too long; max key length is 767 bytes'问题修复
date: 2023-04-06 17:07 +0800
---

今天在尝试搭建 Rails 项目的测试环境时遇见了一个 `Mysql2::Error: Specified key was too long; max key length is 767 bytes` 问题，

经过网上搜索发现是 MySql 会对表的字段名长度做限制，出现这个错误的原因是字段太长了，解决方法也很简单。有两种方法:
1. 修改 Rails 项目的 `config/database.yml` 里的 `encoding: utf8` 设置
``` bash
Unstaged (1)
M config/database.yml
@@ -1,6 +1,6 @@
 default: &default
   adapter: mysql2
-  encoding: utf8mb4
+  encoding: utf8
   pool: 18
@@ -13,6 +13,7 @@ development:
```

2. 使用下面 SQL 语句创建一个数据库
``` SQL
DROP DATABASE IF EXISTS <YOUR_DATABASE_NAME>;
CREATE DATABASE <YOUR_DATABAE_NAME> DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```

