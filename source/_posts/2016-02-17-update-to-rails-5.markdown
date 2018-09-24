---
layout: post
title: "升级Rails 5.2.1"
date: 2016-02-17 17:37
comments: true
categories: "Rails"
---
想要升级到Rails5，你的ruby的版本必须>=2.2.2
可以使用[rvm](http://rvm.io/)方便安装最新的ruby(写这篇文章的时候最新的ruby版本是2.5.1，rails版本是5.2.1，所以就以2.5.1和5.2.1为例)
1. 准备工作安装ruby 2.5.1及rails5.2.1
```sh
# 安装ruby
rvm install 2.5.1
# 创建新的gemset
rvm gemset use 2.5.1@updat_to_rails5 --create
# 安装bundler
gem install bundler
```
2. 修改Gemfile文件，旧内容如下
```ruby
source 'https://rubygems.org'
gem 'rails', '4.2.5'
.
.
.
```
将`gem 'rails', '4.2.5'`修改成`gem 'rails', '5.2.1'`
```ruby
source 'https://rubygems.org'
gem 'rails', '5.2.1'
.
.
.
```
3. 更新rails插件
```sh
bundle update
```

4. 你会遇到很多类似依赖包的版本错误，解决方法是从`Gemfile`中删除这些版本信息后重复第三步

5. 更新rails应用
```
rails app:update
```
PS: 你可能需要添加一些新的gem
``` ruby
# Support for ActiveSupport and YAML, optimize and cache expensive computations
gem 'bootsnap', require: false
# Listens to file modifications and notifies you about the changes.
gem 'listen'
```
PSS: 你运行`rake db:migrate`可能会碰到devise的问题，解决方法是先注释`config/routes.rb`文件里的devise_for和`models/user.rb`文件里的代码。运行`rake db:migrate`之后再将这些代码修改回来。