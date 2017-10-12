---
layout: post
title: "你也许还不知道的关于Rakefiles的特性"
date: 2016-12-10 23:07:38 +0800
comments: true
categories: "Rails"
---

### 任务链

``` ruby
namespace :db do
  task :environment => :environment do
    puts "db:environment"
  end

  task :migrate => :environment do
    puts "db:migrate"
  end
end

task :environment do
  puts "environment"
end
```

当你运行`rake db:environment`时，`db:environment`和`environment`两个任务都会被执行

### 具有状态的任务

``` ruby
LOGGER = Logger.new

task :verbose do
  LOGGER.level = Logger::DEBUG
end

task :deploy do
  LOGGER.debug(:environment){ENV}
# ...
end
```

当你运行`rake verbose deploy`，会输出更多的debug信息

### 可以同时的任务

``` ruby
task :deploy do
  puts "Deploy 1"
end

task :deploy do
  puts "Deploy 2"
end
```

当执行`rake deploy`两个任务都会被执行