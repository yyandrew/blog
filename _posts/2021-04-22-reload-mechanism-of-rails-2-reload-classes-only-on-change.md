---
layout: post
title: Rails自动加载运行机制之reload_classes_only_on_change
date: 2021-04-22 12:52 +0800
categories: 'Rails'
---

下面的源代码都是来自Rails 5.2.4.4。

Rails是通过`cache_classes`和`reload_classes_only_on_change`配置(在development环境下默认是cache_classes为false，即开启自动加载；reload_classes_only_on_change为true)是否开启自动刷新代码功能。

### reload_classes_only_on_change

从名字就可以猜到这个设置是告诉Rails代码更新操作只发生在有文件修改时。比如你修改了一个controller或者一个model，它们都会触发reload。

你可以在[这里](https://github.com/rails/rails/blob/4dcc5435e9569e084f6f90fcea6e7c37d7bd2b4d/railties/lib/rails/application/finisher.rb#L149)找到相关代码。

下面是一个initializer，它会在你启动，初始化Rails应用时候运行。Rails应用通常可以通过`bin/rails server`或者`bin/rails console`命令启动。

```ruby
  initializer :set_clear_dependencies_hook, group: :all do |app|
    callback = lambda do
      ActiveSupport::DescendantsTracker.clear
      ActiveSupport::Dependencies.clear
    end

    if config.cache_classes
      app.reloader.check = lambda { false }
    elsif config.reload_classes_only_on_change
      app.reloader.check = lambda do
        app.reloaders.map(&:updated?).any?
      end
    else
      app.reloader.check = lambda { true }
    end

    if config.reload_classes_only_on_change
      reloader = config.file_watcher.new(*watchable_args, &callback)
      reloaders << reloader

      # Prepend this callback to have autoloaded constants cleared before
      # any other possible reloading, in case they need to autoload fresh
      # constants.
      app.reloader.to_run(prepend: true) do
        # In addition to changes detected by the file watcher, if routes
        # or i18n have been updated we also need to clear constants,
        # that's why we run #execute rather than #execute_if_updated, this
        # callback has to clear autoloaded constants after any update.
        class_unload! do
          reloader.execute
        end
      end
    else
      app.reloader.to_complete do
        class_unload!(&callback)
      end
    end
  end
```

这个intializer会创建一个可调用的对象lambda，它们用于清除内存中已有的常量，下一篇会介绍这两个方法。我们先看看在ruby里`lambda`怎么使用。

我们可以通过`lambda {}`创建一个可调用对象，将其保存在一个常量里，需要的时候使用`call`调用执行，可以达到延迟执行的效果。

```ruby
callback = lambda { puts "I'm a lambda!"}
callback.call # => I'm a lambda!
```

我们还可以通过`&`将lambda对象当作参数传给方法，在方法体调用`call`执行它。

```ruby
callback = lambda { puts "I'm a lambda!" }

def test(&c)
  c.call
end

test(&callback) # => I'm a lambda!
```

这个initializer使用了&传递lambda的方法。

接下来文件监视器`file_watcher`被创建，用来监控`autoload_paths`包含的文件（参数`watchable_args`是由`autoload_paths`以及`schema.rb`、`structure.sql`等文件组成的)。

在传递参数时。`*`操作符可以将参数列表转换成数组。
```ruby
def m(*a)
  puts a.is_a?(Array) # => true
end
m(1, 2, 3)
```

如果`reload_classes_only_on_change`为true时使用file_watcher(Development环境使用`ActiveSupport::EventedFileUpdateChecker`)来监控文件是否有变化，如果有变化`ActiveSupport::EventedFileUpdateChecker#execute`会调用`callback`这个lambda来清空所有的常量(constant)；

如果`reload_classes_only_on_change`为false则通过`to_complete`钩子在每次请求之后调用`callback` 这个lambda来清空所有的常量(constant)。

总结：

1. `initializer`在每次启动Rails应用时运行；
2. 使用`&`操作符可以将lambda传递给方法；
3. `*`操作符可以将方法的参数列表转换成数组；
4. 使用`ActiveSupport::EventedFileUpdateChecker`可以监控文件的变化