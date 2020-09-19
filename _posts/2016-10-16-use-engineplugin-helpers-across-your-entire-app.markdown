---
layout: post
title: "在Rials项目中使用Engine插件提供的helper方法"
date: 2016-10-16 18:46:38 +0800
comments: true
categories: "Rails"
---
当你有Rails项目引入一个engine插件之后，可能会有想在整个Rails项目中能够共享这个插件的一些helper方法。

比如你在你的项目中使用了[spree](https://github.com/spree/spree)。
``` ruby Gemfile
# your code

gem 'spree', '~> 3.1.0'
gem 'spree_auth_devise', '~> 3.1.0'
gem 'spree_gateway', '~> 3.1.0'

# your code
```

你想你的Rails项目使用和插件`spree`相同的header。

你需要在你的Rails项目`app/view/application.html.haml`中加入`= render partial: 'spree/shared/header'`代码。

不出意外你会收到这样的错误

``` irc
undefined local variable or method `logo' for #<#<Class:0x007f9602fa68c0>:0x007f9602fa51f0>`
```

这是因为`spree/shared/header`,用到了插件`spree`的一些helper方法。

为了在Rails项目中引入`spree`的方法，我们可以在application_controller中添加下面的代码

``` ruby app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # your code

  helper Spree::BaseHelper

  # your code
end
```

是不是很简单！！！赶紧试试吧 :)
