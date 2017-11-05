---
layout: post
title: "Rails小抄"
date: 2016-06-07 11:48:46 +0800
comments: true
categories: "Rails"
---

## Rails new命令行的参数

  > 参数：
  >>  * `--skip-bundle` \# 跳过bundle
  >>
  >>  * `--database=postgresql/mysql` \# 指定数据库

## Generate命令
  > scaffold
  >> `rails g scaffold Client name:string`

  > controller
  >> `rails g controller Client`

  > model
  >> `rails g model Client name:string`

## 保留小数点后2位，再分组统计数量

```ruby
Company.where('employee_growth IS NOT NULL').select('round(employee_growth::numeric, 2) AS round_employee_growth').order("round_employee_growth_numeric_2 ASC").group('round(employee_growth::numeric, 2)').count('round(employee_growth::numeric, 2)')
```

## 将where查询语句的结果按照数组的顺序排序

```ruby
uids = User.order('created_at DESC').map &:id
Blog.where(uid: uids).order("position(id::text in '#{uids.join(',')}')")
```
