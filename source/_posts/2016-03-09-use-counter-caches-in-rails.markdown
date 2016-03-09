---
layout: post
title: "在Rails中使用Counter Caches"
date: 2016-03-09 17:28
comments: true
categories: "Rails"
---
Rails提供了一个非常有用的功能-Counter caches。下面我们通过一个例子来说明一下这个功能炫酷之处。
假如有`city`和`company`两个模型，它们的关系如下所示
```ruby
class Company < ActiveRecord::Base
  belongs_to :city
end
```
```ruby
class City < ActiveRecord::Base
  has_many :companies
end
```
如果我们想要知道一个城市有多少公司, 那这时候就是用counter caches的最佳的时机了，通过下面3步就可以轻松实现这个计数功能。

1.修改companies模型中的`belongs_to`代码
```ruby
belongs_to :city, :counter_cache => true
```

2.创建一个新的migration用来向cities表中添加companies_count字段
```sh
rails g migration AddCompaniesCountToCities companies_count:integer
rake db:migrate
```

3.在`rails console`中运行下面命令更新`companies_count`的值
```ruby
City.all.each { |city| City.reset_counters(city.id, :companies)  }
```

然后`cities`表中的`companies_count`就会随着`cities`和`companies`表关系改变自动更新了。
