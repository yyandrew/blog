---
layout: post
title: Ruby内置单例模式是如何实现的
date: 2020-10-20 16:05 +0800
category: Ruby
---

注意：这里提到的单例模式(Singleton Pattern)跟元编程里里的单件类(Singleton Class)是完全不一样的东西。

单例模式保证可以保证调用一个类的`instance`时候确保始终只有一个实例被创建。在`Sidekiq::CLI.instance` 方法里有用到这个功能。这意味着无论你执行多少次`bundle exec sidekiq`，只会产生一个CLI实例。

在开始探索实现原理之前，我们先看看单例模式是如何使用的。先看代码：

```ruby 
require 'singleton'
class Foo
  include Singleton
end
Foo.instance == Foo.instance # => true
```

由于`Singleton`能够被包含，所以它应该是个module。从[源代码](https://github.com/ruby/ruby/blob/18cecda46e427362fa3447679e5d8a917b5d6cb6/lib/singleton.rb#L94)可以看出它确实是个module。我们再看看`instance`这个方法，它应该产生的一个Foo的实例。但是它是在哪里定义的呢？我们可以通过[irb][https://github.com/ruby/irb]查看`instance`在哪里定义。

```
irb(main):007:0> Foo.method(:instance).source_location
=> ["/Users/andrew/.asdf/installs/ruby/2.7.1/lib/ruby/2.7.0/singleton.rb", 121]
```

从最后一行输出可以看出它是被定义在`singleton.rb`。

我们可以用VSCode打开本地或者[github上的ruby代码库](https://github.com/ruby/ruby/blob/18cecda46e427362fa3447679e5d8a917b5d6cb6/lib/singleton.rb#L123)

这个方法的完整代码

```ruby 
def instance # :nodoc:
  # 如果已经创建过单例模式的实例，则直接返回此实例
  return @singleton__instance__ if @singleton__instance__
  @singleton__mutex__.synchronize {
    # 如果已经创建过单例模式的实例，则直接返回此实例
    return @singleton__instance__ if @singleton__instance__
    # 创建一个实例
    @singleton__instance__ = new()
  }
  # 返回这个实例
  @singleton__instance__
end
```

这个方法`@singleton__mutex__`这个对象比较难理解，看起来像是某种多线程锁。

这个对象是在[`__init__`方法](https://github.com/ruby/ruby/blob/18cecda46e427362fa3447679e5d8a917b5d6cb6/lib/singleton.rb#L144)里被创建的。完整代码如下：

```ruby 
def __init__(klass) # :nodoc:
  # 为实例变量@singleton__instance__赋值
  klass.instance_eval {
    @singleton__instance__ = nil
    # 创建一个线程互斥锁，
    # 如果要深入了解互斥锁，可以查看文档https://ruby-doc.org/core-2.5.0/Mutex.html
    @singleton__mutex__ = Thread::Mutex.new
  }
  klass
end
```

现在我们已经搞清楚`instance`是如何做只产生一个实例了。但是还有另外一个特性还有必要补充一下。这个特性是当一个类包含一个`Singleton`模块之后，你不能使用`new`创建实例了(这个说法不准确，因为你还是可以通过`Foo.send(:new)`来成功创建实例)，比如下面的代码是不能工作的：

```shell
irb(main):008:0> Foo.new
Traceback (most recent call last):
        4: from /Users/andrew/.asdf/installs/ruby/2.7.1/bin/irb:23:in `<main>'
        3: from /Users/andrew/.asdf/installs/ruby/2.7.1/bin/irb:23:in `load'
        2: from /Users/andrew/.asdf/installs/ruby/2.7.1/lib/ruby/gems/2.7.0/gems/irb-1.2.3/exe/irb:11:in `<top (required)>'
        1: from (irb):8
NoMethodError (private method `new' called for Foo:Class)
```

错误信息很明显，`new`被转换成了私有方法。很显然这是`Singleton`帮我们做的。问题是如何做到的呢？答案就是：`Singleton`通过[`included`](https://github.com/ruby/ruby/blob/18cecda46e427362fa3447679e5d8a917b5d6cb6/lib/singleton.rb#L162)钩子将`new`方法转换成的私有方法。完整代码如下：

```ruby
def included(klass)
  super
  # 将类公共方法`new`转换成私有方法
  klass.private_class_method :new, :allocate
  klass.extend SingletonClassMethods
  Singleton.__init__(klass)
end
```



