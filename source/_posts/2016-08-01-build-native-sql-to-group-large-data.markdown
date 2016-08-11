---
layout: post
title: "构建sql统计n天内的数据"
date: 2016-08-01 18:39:34 +0800
comments: true
categories: "Rails"
---
## 下面以一周内每天的用户注册量为例子说明怎么创建这样的sql

#### 构建sql的方法

```ruby app/models/user.rb
class User < ActiveRecord::Base
  def self.group_by_days(days)
    sql = "SELECT "
    (1..days).each do |i|
      date = (days - i + 1).days.ago.utc

      sql += "COUNT(NULLIF((created_at >= '#{date.at_beginning_of_day}' AND created_at < '#{date.end_of_day}'), FALSE)) AS day#{date.strftime('%Y_%m_%d')}#{',' unless i == days} "
    end
    sql += "FROM users"

    self.connection.execute(sanitize_sql(sql))[0]
  end
end
```

#### 使用方法
```ruby
User.group_by_days(7)
```

#### 执行结果
``` irc
[28] pry(main)> User.group_by_days(7)
   (0.8ms)  SELECT COUNT(NULLIF((created_at >= '2016-08-04 00:00:00 UTC' AND created_at < '2016-08-04 23:59:59 UTC'), FALSE)) AS day2016_08_04, COUNT(NULLIF((created_at >= '2016-08-05 00:00:00 UTC' AND created_at < '2016-08-05 23:59:59 UTC'), FALSE)) AS day2016_08_05, COUNT(NULLIF((created_at >= '2016-08-06 00:00:00 UTC' AND created_at < '2016-08-06 23:59:59 UTC'), FALSE)) AS day2016_08_06, COUNT(NULLIF((created_at >= '2016-08-07 00:00:00 UTC' AND created_at < '2016-08-07 23:59:59 UTC'), FALSE)) AS day2016_08_07, COUNT(NULLIF((created_at >= '2016-08-08 00:00:00 UTC' AND created_at < '2016-08-08 23:59:59 UTC'), FALSE)) AS day2016_08_08, COUNT(NULLIF((created_at >= '2016-08-09 00:00:00 UTC' AND created_at < '2016-08-09 23:59:59 UTC'), FALSE)) AS day2016_08_09, COUNT(NULLIF((created_at >= '2016-08-10 00:00:00 UTC' AND created_at < '2016-08-10 23:59:59 UTC'), FALSE)) AS day2016_08_10 FROM users
=> {"day2016_08_04"=>"0",
 "day2016_08_05"=>"0",
 "day2016_08_06"=>"0",
 "day2016_08_07"=>"0",
 "day2016_08_08"=>"0",
 "day2016_08_09"=>"0",
 "day2016_08_10"=>"0"}
```
