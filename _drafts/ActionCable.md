### 为什么要阅读源码

1. 学习设计思想
2. 加深知识理解
3. 提高编程水平
4. 应对面试等等

### 如何选择项目

1. 学习常用框架源码，activerecord/activestorage/actioncable
2. 学习常用库源码：sidekiq，puma
3. 学习github上高质量项目源码：discourse

### 怎么阅读源码

1. 站在巨人的肩膀，提前阅读一些相关优质文章
2. 汇总阅读过程中或者工作中遇到的问题
3. 明确阅读源码的主线或切入点

### 如何阅读Action Cable?

##### 走进Action Cable

Action Cable将Websocket与Rails应用的其余部分无缝集成。有了Action Cable，我们就可以用Ruby语言，以Rails的风格实现实时功能，并且保持高性能和可扩展性。

##### 探索Action Cable的特性

在确认了研究对象之后，我们去探索它的特性。特性可以从REMAME里获得。

1. 集成了websocket
2. 和Rails应用的其它部分无缝集成，也就是可以使用Rails的其它模块，比如：Active Record。
3. 使用发布/订阅在服务器和多个客户端之间通信。
4. 提供了客户端JavaScript框架和服务器端Ruby框架。

##### 深入理解某个特性

以**发布/订阅**这个切入点为例，首先我们会接触到**消息队列**的概念。发布/订阅(pub/sub)是指在消息队列中，信息发送者(发布者)把数据送给某个接收者(订阅者)，而不必单独指定接收者。Action Cable通过发布/订阅的方式在服务器和多个客户端之间通信。

###### 创建服务器端频道

频道把已发布的内容发送给订阅者，是通过所谓的'流'机制实现的

````ruby
# app/channels/web_notifications_channel.rb
class WebNotificationsChannel < ApplicationCable::Channel
  def subscribed
    stream_for current_user
  end
end
````

###### 客户端频道订阅

订阅频道的用户，称为订阅者。用户创建的连接称为（频道）订阅。订阅基于用户（订阅者）发送的标识符创建，收到的消息将被发送到这些订阅

```coffeescript
# app/assets/javascripts/cable/subscriptions/web_notifications.coffee
# 客户端假设我们已经获得了发送 Web 通知的权限
App.cable.subscriptions.create "WebNotificationsChannel",
  received: (data) ->
    new Notification data["title"], body: data["body"]
```

`ActionCable::Channel::Streams#stream_from`

###### 服务器端发布消息(内容)

```ruby
# 在应用的某个部分中调用，例如 NewCommentJob
WebNotificationsChannel.broadcast_to(
  current_user,
  title: 'New things!',
  body: 'All the news fit to print'
)
```

` ActionCable::Channel::Base.broadcast_to`来自 `actioncable-6.0.3.4/lib/action_cable/server/broadcasting.rb:11`

broadcast_to 触发`ActiveSupport::Notifications.instrument`，`ActiveSupport::Notifications.instrument`来自`activesupport-6.0.3.4/lib/active_support/notifications.rb:178` 

`instrument`触发了`pubsub`方法

```

```



redis适配器使用的是[redis的pubsub特性](https://redis.io/topics/pubsub#database-amp-scoping)

postgresql适配器使用的是[postgresql的NOTIFY/LISTEN特性](https://www.postgresql.org/docs/9.0/sql-notify.html)