---
layout: post
title: "includes多层子Model方法"
date: 2015-09-01 17:06
comments: true
categories: "Rails"
---
众所周知includes可以消除N+1次查询，提高查询速度。但是如果一个Model有多层子Model该怎么查询呢？
可以用下面方法
```ruby
@app = Application.includes(:first_submodel,
                            {:second_submodel => [:first_sub_submodel, :second_sub_submodel]},
                            {:third_submodel  => :third_sub_submodel})
```
