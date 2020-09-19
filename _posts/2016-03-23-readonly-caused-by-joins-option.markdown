---
layout: post
title: "由joins造成的readonly错误的解决方法"
date: 2016-03-23 17:42
comments: true
categories: "Rails"
---
今天碰到一个非常奇怪的问题,下面是错误的重新步骤。
``` bash
pry(main)> product = Product.query_by_name("shanghai").first
pry(main)> product.update_attribute(:is_delete, true)
  (0.1ms)  BEGIN
  (0.1ms)  ROLLBACK
ActiveRecord::ReadOnlyRecord: ActiveRecord::ReadOnlyRecord
from /Users/andrew/.rvm/gems/ruby-1.9.3-p551@litool/gems/activerecord-3.2.13/lib/active_record/persistence.rb:347:in `create_or_update'`
```
经过搜索之后发现这样的说明
> Introduce read-only records. If you call object.readonly! then it will mark the object as read-only and raise ReadOnlyRecord if you call object.save. object.readonly? reports whether the object is read-only. Passing :readonly => true to any finder method will mark returned records as read-only. The :joins option now implies :readonly, so if you use this option, saving the same record will now fail. Use find_by_sql to work around.

原来是在使用`joins`时会强制使用`readonly`

``` ruby
class Product < ActiveRecord::Base
  ...
  scope :query_by_name, lambda { |name|
      joins("LEFT JOIN ...").where('...')
      }
  ...
end
```
其实这个问题有两种解决方法
1.我们只需要在scope代码添加`readonly(false)`就可以了
```ruby
class Product < ActiveRecord::Base
  ...
  scope :query_by_name, lambda { |name|
      joins("LEFT JOIN ...").uniq.readonly(false)
      }
  ...
end
```
2.就像上面官方的文档用`find_by_sql`替换`joins`
