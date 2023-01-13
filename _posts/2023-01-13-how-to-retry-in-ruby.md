---
layout: post
title: ruby里如何进行重试
date: 2023-01-13 09:48 +0800
---

ruby 里提供一个 retry 的方法可以进行异常之后的重试操作，一般会用在网络请求报错重试。

下面是一段示例代码：

``` ruby
puts 'This will not retry'
begin
  retries ||= 0
  puts "retries at #{retries}"
  # The third try will success
  raise 'An exception happened' if retries < 2

  # All tries are failed
  # raise 'An exception happened'
rescue Exception => e
  sleep 1 # delay 1 second
  retry if (retries += 1) < 3 # retry 3 times

  puts "error: #{e}" # print error message
end
```

根据代码运行结果可以看出下面的规律
1. 只有 `begin` 和 `rescue` 里的代码会被重新执行，`begin` 和 `rescue` 之外的代码不会被重试;
2. 如果重试成功 `rescue` 里的代码就不会再被执行
