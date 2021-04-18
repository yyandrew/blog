---
layout: post
title: Rails自动加载运行机制之开篇
date: 2021-04-17 20:52 +0800
categories: 'Rails'
---

这是个系列文章，这篇文章只是描述自动加载的大致的流程

我们知道在Rails项目中修改一个model或者controller之后不需要重新启动rails server，rails能够帮我们自动刷新代码，让新代码自动生效，从而加快我们的开发效率。比如有在`app/models/user.rb`里添加一个新的方法，然后在`app/views/users/show.html.erb`文件调用这个方法，我们只用将修改保存，再刷新页面就可以测试修改了。

下面我们来看看Rails是如何实现这一功能的。

先说一下代码重新加载的大致的流程：

1. 客户端发生一个请求，服务器使用`ActionDispatch::Reloader`执行一些回调方法；
2. 其中的一个回调方法会使用`ActiveSupport::EventedFileUpdateChecker`来检查是不是有文件发生修改；
3. 如果有修改发生，则清空所有代码，比如类常量会被清除
4. 请求被传递到middleware组件栈里的下一个组件
5. 当这个middleware组件发现常量被清除了，它会调用`ActiveSupport::Dependencies`重新加载这些常量。

之后几篇会根据这个流程结合源代码详细说明工作原理。