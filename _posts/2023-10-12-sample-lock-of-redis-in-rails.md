---
layout: post
title: 使用redis实现一个简单的防抖功能
date: 2023-10-12 11:08 +0800
---
使用 Redis 的 nx 属性可以限制执行频率。

下面的代码实现了 6 秒只执行一次查找用户的逻辑。

```ruby
def lock
  email = 'foo@bar'
  key = "lock/#{email}"
  if $redis.set(key, 1, nx: true, ex: 6.seconds)
    user = User.find_by(email: email)
    return if user

    User.create!(email: email)
  end
end

```
