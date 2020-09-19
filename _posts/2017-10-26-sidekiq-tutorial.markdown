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

3, 任务保存在哪里?

如果是延时任务：比如`GuestsCleanupJob.set(wait: 1.week).perform_later(guest)`,它会保存到redis的有序集合(zset)里。

如果是立即执行任务，比如`GuestsCleanupJob.perform_later(guest)`，它会被保存在列表(list)里。

源码：

``` ruby
# https://github.com/mperham/sidekiq/blob/ba6fcd66ba60fc9b13ef7d1a431701667bfd18f9/lib/sidekiq/client.rb#L196
def atomic_push(conn, payloads)
  if payloads.first.key?("at")
    # 定时任务
    conn.zadd("schedule", payloads.map { |hash|
      at = hash.delete("at").to_s
      [at, Sidekiq.dump_json(hash)]
    })
  else
    # 立即执行任务
    queue = payloads.first["queue"]
    now = Time.now.to_f
    to_push = payloads.map { |entry|
      entry["enqueued_at"] = now
      Sidekiq.dump_json(entry)
    }
    conn.sadd("queues", queue)
    conn.lpush("queue:#{queue}", to_push)
  end
end
```
4, 如何消费队列里的job
运行`bundle exec sidekiq`就可以去执行队列里的job了。它会执行`Sidekiq::CLI.instance`方法.
先看看sidekiq脚本：
```
#!/usr/bin/env ruby
# 省略部分不重要代码
...
begin
  # 创建一个CLI对象实例
  # `.instance`为了确保只有一个实例
  cli = Sidekiq::CLI.instance
  # 1. 解析配制文件并赋值
  # 2. 设置日志
  # 3. 验证`concurrency`及`timeout`选项的值是否合法
  cli.parse

  integrate_with_systemd

  # 验证当前环境是否满足运行环境
  cli.run
rescue => e
  raise e if $DEBUG
  if Sidekiq.error_handlers.length == 0
    STDERR.puts e.message
    STDERR.puts e.backtrace.join("\n")
  else
    cli.handle_exception e
  end

  exit 1
end
```
`cli.run`会调用`cli.launch`方法.`launch`最终会调用`Sidekiq::Scheduled::Enq#enqueue_jobs`方法。
```
# https://github.com/mperham/sidekiq/blob/3f54edb4497ee727b947effaff46d88302270a84/lib/sidekiq/scheduled.rb#L12
def enqueue_jobs(now = Time.now.to_f.to_s, sorted_sets = SETS)
  # A job's "score" in Redis is the time at which it should be processed.
  # Just check Redis for the set of jobs with a timestamp before now.
  Sidekiq.redis do |conn|
    sorted_sets.each do |sorted_set|
      # Get next items in the queue with scores (time to execute) <= now.
      until (jobs = conn.zrangebyscore(sorted_set, "-inf", now, limit: [0, 100])).empty?
        # We need to go through the list one at a time to reduce the risk of something
        # going wrong between the time jobs are popped from the scheduled queue and when
        # they are pushed onto a work queue and losing the jobs.
        jobs.each do |job|
          # Pop item off the queue and add it to the work queue. If the job can't be popped from
          # the queue, it's because another process already popped it so we can move on to the
          # next one.
          if conn.zrem(sorted_set, job)
            Sidekiq::Client.push(Sidekiq.load_json(job))
            Sidekiq.logger.debug { "enqueued #{sorted_set}: #{job}" }
          end
        end
      end
    end
  end
end
```
可以看到这个方法遍历了所有的队列集合通过Redis的`zrangebyscore`方法攻取所有的当前时间之前的job。
