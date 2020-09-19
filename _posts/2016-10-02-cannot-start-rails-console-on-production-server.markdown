---
layout: post
title: "cannot start rails console on production server"
date: 2016-10-02 07:21:37 +0800
comments: true
categories: "Rails"
---

最近部署新项目到生产环境后，运行`bundle exec rails c -e production`
出现下面错误

``` sh
Usage:
  rails new APP_PATH [options]

Options:
  -r, [--ruby=PATH]                                      # Path to the Ruby binary     of your choice

..... yada yada, shows the rest of the rails options (oddly enough does not show 'c' or 'console' as options?)
```
capistrano的版本

```
cap -V # Capistrano Version: 3.6.1 (Rake Version: 11.3.0)
```

生产环境

```
rvm -v # rvm 1.26.11 (latest) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
bundle exec rails -v # Rails 5.0.0.1
```

经过google发现这个问题是因为`current`目录下没有'bin/rails'造成的。

难道是`.gitignore`文件设置了忽略了`bin`文件夹？一检查，果然有

``` sh .gitignore
log/*
tmp/*
config/database.yml
config/secrets.yml
bin/*
.byebug_history
*.swp
public/media/*
```

没有什么好说的，先删掉`bin/*`行。
接下来你也许还要检查一下你的`deploy.rb`文件`set :linked_dirs`是不是有`bin`的字样，如果有也请删除`bin`
``` sh 有问题的deploy.rb设置
set :linked_dirs, fetch(:linked_dirs, []).push('bin', 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system', 'public/assets', 'public/media')
```

``` sh 修改后的deploy.rb设置
set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system', 'public/assets', 'public/media')
```

push代码之后，重新部署一下应该就没有问题了。
