---
layout: post
title: "sti and form_for"
date: 2015-03-21 09:34
comments: true
categories: "Rails"
---

Rails的form_for会根据你的object的class名称自动产生URL。

user.rb
```ruby
class User < ActiveRecord::Base
end
```
users_controller.rb
```ruby
def edit
  @user = User.find(1) # user对象
end
```
views/users/_form.html.haml
```ruby
= form_for(@user) do |f|
  = f.text_field :name
```
这时这个表单会将数据post到users_path去创建一个新的user或者putch到user_path(@user)更新user信息。代码到现在是没有任何问题。
```ruby
class ForntEndUser < User
end
```
users_controller.rb
```ruby
def edit
  @user = User.find(2) # front_end_user对象
end
```
views/users/_form.html.haml
```ruby
= form_for(@user) do |f|
  = f.text_field :name
```
此时表单会将数据post到front_end_users_path。但是我们还没有定义front_end_users_path。
```ruby
undefined method `front_end_users_path' for #<#<Class:0xc052150>:0xc050404>
```
在[form的文档](http://guides.rubyonrails.org/form_helpers.html)已经告诫我们STI的自动识别功能是不可靠的。
解决的方法很简单。[#becomes](http://apidock.com/rails/ActiveRecord/Persistence/becomes)能够装将front_end_user转换成user
```ruby
= form_for(@user.becomes(User)) do |f|
  = f.text_field :name
```

