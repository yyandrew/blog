---
layout: post
title: "Sidekiq入门教程"
date: 2017-10-26 12:00:22 +0800
comments: true
categories: 'Rails'
---

1, 列出所有出错的jobs

``` ruby
query = Sidekiq::Failures::FailureSet.new.map(&:item)
query.map { |item| item['class'] }.uniq # Gives you a sense of what failed
query.map { |item| item['wrapped'] }.uniq # Gives you the job's original class
query.map { |item| item['queue'] }.uniq # Gives you a name of each of the queues
query.map { |item| item['args'] }.uniq # This would give you all the argument hashes for all failures.  
```

2, 删除所有名字是SessionJob出错的jobs

```ruby
Sidekiq::Failures::FailureSet.new.select {|item| item['wrapped'] == 'SessionJob' }.map &:delete
```

Tips: [query name](https://github.com/mperham/sidekiq/wiki/API#named-queues)
