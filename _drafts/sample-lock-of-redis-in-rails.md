---
layout: post
title: sample lock of redis in rails
---

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
