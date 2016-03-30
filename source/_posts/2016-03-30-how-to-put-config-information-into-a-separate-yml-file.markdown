---
layout: post
title: "怎么将公用的配置信息移动的单独的yml文件中"
date: 2016-03-30 18:48
comments: true
categories: "Rails"
---
1.创建`config/aws.yml`文件
```ruby
common: &common
  api_key: AWS_ACCESS_KEY_ID
  api_secret: AWS_SECRET_KEY
  product:
    request_url: REQUEST_URI

product:
  <<: *common

development:
  <<: *common

test:
  <<: *common
```
2.修改`application.rb`文件使得`config/aws.yml`能够自动被加载
```ruby
AWS = YAML.load(File.read(File.expand_path('../aws.yml', __FILE__))).fetch(Rails.env)
```
做完这样之后你就可以有项目的任意位置使用`AWS[:api_key]`读取`api_key`的值了
