---
layout: post
title: "利用正则表达式判断是不是合法邮箱"
date: 2015-11-07 19:54
comments: true
categories: "Rails"
---
分享一个判断邮箱的方法(来自[email_validator](https://github.com/balexand/email_validator/blob/738d73a2b34c59c7019f0d4c67168884c96b09bd/lib/email_validator.rb))
```ruby
class EmailValidator
  def self.regexp
    name_validation = "^@\\s"
    /\A\s*([#{name_validation}]{1,64})@((?:[-\p{L}\d]+\.)+\p{L}{2,})\s*\z/i
  end

  def self.valid?(value)
    !!(value =~ regexp)
  end
end
```
使用方法:
```ruby
 EmailValidator.valid?("andrew@example.com") # return true
 EmailValidator.valid?("andrew.com") # return false
```
