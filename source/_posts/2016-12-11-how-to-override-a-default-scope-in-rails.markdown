---
layout: post
title: "怎么重写Rails中的default_scope"
date: 2016-12-11 20:03:19 +0800
comments: true
categories: "Rails"
---

``` ruby
class Employee < ApplicationRecord
  scope :managers, -> { where(role: 'manager') }
  default_scope { managers.order(name: :desc) }
end

Employee.order(created_at: :desc) # 按照created_at的排序逻辑会被加在name排序之后
Employee.unscoped.order(created_at: :desc) # 所有的scope都会被忽略，包含managers
Employee.reorder(created_at: :desc) # 可以覆盖name的排序，只使用created_at排序，但是效率低

# 正确的方法
Employee.unscope(:order).order(created_at: :desc)
# 或者
Employee.unscope(order: [:name]).order(created_at: :desc)
```
