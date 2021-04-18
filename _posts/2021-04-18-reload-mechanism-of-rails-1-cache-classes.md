---
layout: post
title: Rails自动加载运行机制之cache_classes
date: 2021-04-18 12:52 +0800
categories: 'Rails'
---

下面的源代码都是来自Rails 5.2.4.4。

Rails是通过`cache_classes`和`reload_classes_only_on_change`配置(在development环境下默认是cache_classes为false，即开启自动加载；reload_classes_only_on_change为true)是否开启自动刷新代码功能。

### cache_classes

通过搜索我们发现在`Rails::Application::DefaultMiddlewareStack`里面包含处理`config.cache_classes`的相应的代码。

```ruby
   unless config.cache_classes
        middleware.use ::ActionDispatch::Reloader, app.reloader
   end
```

`middleware`是`ActionDispatch::MiddlewareStack`实例。它的`use`方法如下：

```ruby
def use(klass, *args, &block)
  middlewares.push(build_middleware(klass, args, block))
end
```

`ActionDispatch::Reloader`是一个Rack middleware。它的作用是每当有代码修改发生自动更新代码，确保每个HTTP请求能够使用最新的代码，

`app.reloader`是`ActiveSupport::Reloader` 的子类。其赋值语句为：

```ruby
@reloader          = Class.new(ActiveSupport::Reloader)
```

ps:使用`Class.new`可以创建一个匿名类，它的参数是这个匿名类的父类。它定义了一些callbacks。在自动更新代码时被调用执行。

通过`middleware.use`方法使得middleware栈多了一个`::ActionDispatch::Reloade`的middleware。

middleware有什么作用呢？middleware是位于客户端和服务端之间的组件，我们可以使用它来处理流入、流出的客户端请求数据或者服务端的响应数据。

怎么自定义一个middleware?

```ruby
class MyMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    puts "MiddlewareOne reporting in!"
    puts "The app is #{@app}"
    status, headers, body = @app.call(env)
    [status, headers, body]
  end
end
```

在`Rack`里使用middleware

```ruby
// config.ru
class HandleClass
  def self.call(env)
    [200, { 'Content-Type' => 'text/html'}, ['Hello middle']]
  end
end
use MyMiddleware
run HandleClass
```

使用命令`rackup config.ru`运行`config.ru`，启动WEBrick服务器。访问http://127.0.0.1:9292.。可以看到日志输出类似：

```
MiddlewareOne reporting in!
The app is HandleClass
```

说明我们的middleware起作用了。

总结：

1. `ActionDispatch::MiddlewareStack#use`可以向middleware栈添加新的middleware；
2. 如果`cache_classes`为`false`时`ActiveSupport::Reloader`这个middleware会被添加到middleware栈里；
3. 使用`Class.new`可以创建一个匿名类，它的参数是这个匿名类的父类；
4. middleware是位于客户端和服务端之间的组件，我们可以用它来处理请求或者响应数据；
5. 使用Rack可以开发web应用。`rackup`命令可以运行Rack应用。