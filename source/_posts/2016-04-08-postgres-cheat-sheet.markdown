---
layout: post
title: "postgres小抄"
date: 2016-04-08 11:14
comments: true
categories: "postgres"
---
### 进入psql终端
```sh
psql -U username dbname
```
### 表操作
```sh
dbname=# \d tablename; # 显示表的结构
dbname=# ALTER TABLE tablename ADD COLUMN newcolumn integer; # [字段类型](http://www.postgresql.org/docs/9.3/static/datatype.html#DATATYPE-TABLE)
dbname=# ALTER TABLE tablename DROP COLUMN oldcolumn; # 删除字段
dbname=# ALTER TABLE tablename ALTER COLUMN last_updated_at SET DEFAULT now(); # 设置默认值
```
