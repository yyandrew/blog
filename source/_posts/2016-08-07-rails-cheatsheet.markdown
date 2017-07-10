---
layout: post
title: "Rails cheatsheet"
date: 2016-06-07 11:48:46 +0800
comments: true
categories: "Rails"
---
## rails new app_name

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

## Round employee_growth to two decimal and then group

```ruby
Company.where('employee_growth IS NOT NULL').select('round(employee_growth::numeric, 2) AS round_employee_growth').order("round_employee_growth_numeric_2 ASC").group('round(employee_growth::numeric, 2)').count('round(employee_growth::numeric, 2)')
```
