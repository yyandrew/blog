---
layout: post
title: JSON类型字段导致MySQL逆向工程失败
date: 2020-10-19 11:34 +0800
category: MySQL
---

### 问题现象

```
ERROR: (3, 12) "json" is not valid at this position, expecting BIGINT, BINARY, BIT, BLOB, BOOLEAN, BOOL, ...
ERROR: (3, 15) "json" is not valid at this position, expecting BIGINT, BINARY, BIT, BLOB, BOOLEAN, BOOL, ...
ERROR: (3, 10) "json" is not valid at this position, expecting BIGINT, BINARY, BIT, BLOB, BOOLEAN, BOOL, ...
```

![json-is-not-valid-at-this-position.png](/images/json-is-not-valid-at-this-position.png)

### 问题原因

由于这张表包含3个JSON类型的字段，同时MySQL版本太老，导致逆向工程失败。

JSON字段是在**5.7.8**开始被支持

### 解决方法

首先升级MySQL，然后修改MySQL Workbench的`Prefermance->Modeling->MySQL里的`Default Target MySQL Version:`为升级后的新版本。

比如我本地的MySQL升级之后的版本为**5.7.21**。

```
$ mysql --version
mysql  Ver 14.14 Distrib 5.7.21, for macos10.13 (x86_64) using  EditLine wrapper
```

那么这里应该填入`5.7.21`。

![update-default-target-mysql-version-of-modeling.png](/images/update-default-target-mysql-version-of-modeling.png)

修改保存之后应该就就不会报错了。
