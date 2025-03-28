---
layout: post
title: 使用 redis 的 bitmap 来只输出一次日志
date: 2025-03-28 13:27 +0800
---
### 功能
在 Rails 项目中使用 redis 的 bitmap 来只输出一次日志。

### 应用场景
我们项目需要实现一个监控功能，如果优惠券状态异常需要打印一次日志。
### 实现逻辑
1. 如果优惠券状态异常，redis 的 bitmap 中设置一个值。并且打印日志。
2. 如果优惠券状态正常，需要将 redis 的 bitmap 中的值清除。

#### 异常情况的代码
``` ruby
if !@coupon.is_valid?
    return if $redis.getbit("coupon_error", @coupon.id).to_i.postive?

    $redis.setbit("coupon_error", @coupon.id, 1)
    Rails.logger.warn "优惠券 #{@coupon.id} - #{@coupon.title} 异常!"
end
```

#### 优惠券状态正常后的代码
``` ruby
if @coupon.is_valid?
    $redis.setbit("coupon_error", @coupon.id, 0)
end
```

