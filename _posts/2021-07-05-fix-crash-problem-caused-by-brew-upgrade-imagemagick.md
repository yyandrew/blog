---
layout: post
title: imagemagick升级导致rails s崩溃的解决方法
date: 2021-07-05 16:50 +0800
---
#### 事件起因
起因就是手贱导致 imagemagick 被升级，导致 rails s 崩溃，无法启动 rails 服务器。

是报错信息如下:
```
$ rails s
Traceback (most recent call last):
        32: from bin/rails:4:in `<main>'
        31: from bin/rails:4:in `require'
        30: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/railties-6.1.3.2/lib/rails/commands.rb:18:in `<top (required)>'
        29: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/railties-6.1.3.2/lib/rails/command.rb:50:in `invoke'
        28: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/railties-6.1.3.2/lib/rails/command/base.rb:69:in `perform'
        27: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/thor-1.1.0/lib/thor.rb:392:in `dispatch'
        26: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/thor-1.1.0/lib/thor/invocation.rb:127:in `invoke_command'
        25: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/thor-1.1.0/lib/thor/command.rb:27:in `run'
        24: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/ems/railties-6.1.3.2/lib/rails/commands/server/server_command.rb:135:in `perform'
        23: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/railties-6.1.3.2/lib/rails/commands/server/server_command.rb:135:in `tap'
        22: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/railties-6.1.3.2/lib/rails/commands/server/server_command.rb:138:in `block in perform'
        21: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `require'
        20: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:299:in `load_dependency'
        19: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `block in require'
        18: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `require'
        17: from /Users/andrew/Development/myapp/config/application.rb:17:in `<top (required)>'
        16: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/2.7.0/bundler.rb:174:in `require'
        15: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/2.7.0/bundler/runtime.rb:58:in `require'
        14: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/2.7.0/bundler/runtime.rb:58:in `each'
        13: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/2.7.0/bundler/runtime.rb:69:in `block in require'
        12: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/2.7.0/bundler/runtime.rb:69:in `each'
        11: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/2.7.0/bundler/runtime.rb:74:in `block (2 levels) in require'
        10: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/2.7.0/bundler/runtime.rb:74:in `require'
         9: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/rmagick-4.1.2/lib/rmagick.rb:1:in `<top (required)>'
         8: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `require'
         7: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:299:in `load_dependency'
         6: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `block in require'
         5: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `require'
         4: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/rmagick-4.1.2/lib/rmagick_internal.rb:23:in `<top (required)>'
         3: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `require'
         2: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:299:in `load_dependency'
         1: from /Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `block in require'
/Users/andrew/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/activesupport-6.1.3.2/lib/active_support/dependencies.rb:332:in `require': This installation of RMagick was configured with ImageMagick 7.0.10 but ImageMagick 7.1.0-2 is in use. (RuntimeError)
 g
```
#### 分析原因
从最后一行日志，可以看出原因是: RMagick 是通过 ImageMagick 7.0.10 安装的，但是我已经升级到了 ImageMagick 7.1.0-2，7.0.10 已经不存在了，所以报错了。重新安装 rmagick 可以解决。
#### 解决方法 
1. `bundle exec uninstall rmagick` # 卸载旧的 rmagick
2. `export PKG_CONFIG_PATH="/usr/local/opt/imagemagick@7/lib/pkgconfig"` # 将新版本的配置文件添加到环境变量 PKG_CONFIG_PATH里
3. `pkg-config --libs imagemagick` # 查看配置文件是否设置成功
4. `bundle` # 重新安装 rmagick
