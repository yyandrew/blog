---
layout: post
title: "强制使用Rails的整数类型'type'列实现单表继承"
categories: Rails
date: 2020-09-28 14:09 +0800
---
事情起因：一个老的PHP项目有一个`OrderItems`表，它有一个`type`字段，`type`字段是整数`1`, `2`, `3`。表不同的类型`OrderItemOffer`，`OrderItemProduct`及`OrderItemProductUse`三个不同的类。

在Rails里，想要实现单表继承`type`默认是一个字符串类型。

那么问题就出现了：怎么利用现有的数据表在Rails实现单表继承？

通过查看ActiveRecord的源码。单表继承的实现在[`ActiveRecord::Inheritance`](https://github.com/rails/rails/blob/9a5531bb7c872f5cf25e9ce7f0ce5b51c4b8abfb/activerecord/lib/active_record/inheritance.rb#L37)。

想要通过整数类型`type`实现单表继承需要重写这两个方法：

1. `sti_name` 用来检索`type`的值
2. `find_sti_class `用来将存储在`type`中的值转换为相应的ActiveRecord模型



新建一个`app/models/concerns/inheritance_from_integer_type.rb`，内容如下：

```ruby 
# frozen_string_literal: true

module InheritanceFromIntegerType
  extend ActiveSupport::Concern
  module ClassMethods
    def value_of_type=(value)
      @value = value
    end

    def sti_name
      @value
    end

    def find_sti_class(_type_name)
      self
    end
  end
end
```

将`InheritanceFromIntegerType`引入到对应的model。比如`OrderItemOffer`。

```ruby 
class OrderItemOffer < OrderItem
  self.table_name = 'order_items'
  include InheritanceFromIntegerType
  # 设置需要检索的type的值
  self.value_of_type = 1
end
```

测试结果：

```shell 
[7] pry(main)> OrderItemOffer.limit(10).to_sql
=> "SELECT `order_items`.* FROM `order_items` WHERE `order_items`.`type` = 1 LIMIT 10"
```

从上面输出的SQL查询语句可以看到我们使用的`type = 1`这个条件。
