---
layout: post
title: Group里的gem依赖包不能自动加载到$LOAD_PATHS的解决方案
date: 2023-02-08 09:16 +0800
---

### 问题
运行`rails console`时不能自动进入`pry`环境，`Gemfile`文件里的`pry`系列依赖包如下，放在`test`及`development`这两个组，表示只有在这两个环境下`pry`才有效。
```
group :test, :development do
  gem 'pry'
  gem 'pry-doc'
  gem 'pry-rails'
  gem 'pry-byebug'
end
```
### 排查过程
1. 运行`rails console`进入到`irb`环境，使用`$LOAD_PATH.find {|path| path.match /pry/}`看看`$LOAD_PATHS`是否包含`pry`，结果是`nil`表示`pry`没有自动加载。
2. 退出`rails console`环境到shell，运行`bundle show pry`
3. 百思不得其解，只能上网搜结果了，最后找到[是因为项目目录下包含`.bundle/config`导致的](https://stackoverflow.com/a/26993819)，下面是我的项目的`.bundle/config`内容
![bundle-show-pry.png](/images/bundle-show-pry.png)
### 解决方法
删除掉`.bundle/config`文件，重新运行`bundle`，使用`rails console`发现我们的好帮手`pry`又回来了。
