---
layout: post
title: "升级Rails 5.0.0.beta2"
date: 2016-02-17 17:37
comments: true
categories: "Rails"
---
想要升级到Rails5，你的ruby的版本必须>=2.2.2
可以使用[rvm](http://rvm.io/)方便安装最新的ruby(写这篇文章的时候最新的ruby版本是2.2.3，rails版本是5.0.0.beta2，所以就以2.2.3和5.0.0.beta2为例)
1. 准备工作安装ruby 2.2.3及rails5.0.0.beta2
```sh
rvm install 2.2.3
rvm gemset use 2.2.3@rails5 --create
gem install bundler
bundle install
gem install rails --pre
```
2. 修改Gemfile文件
```ruby
source 'https://rubygems.org'
gem 'rails', '4.2.5'
.
.
.
```
将`gem 'rails', '4.2.5'`修改成`gem 'rails', '5.0.0.beta2'`
```ruby
source 'https://rubygems.org'
gem 'rails', '5.0.0.beta2'
.
.
.
```
3. 更新rails
```sh
bundle update rails
```
