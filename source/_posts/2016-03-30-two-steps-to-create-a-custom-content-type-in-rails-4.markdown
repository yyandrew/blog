---
layout: post
title: "只需两步在Rails4中创建自定义content type"
date: 2016-03-30 17:25
comments: true
categories: "Rails"
---
有时候创建一个自定义的content type是非常有用的，所幸Rails为我们提供了非常简单的实现方法.
下面我们就以自定义一个xls的content type做为例子展现Rails为我们带来的便利吧

1.修改`config/initializers/mime_types.rb`文件
```ruby
Mime::Type.register "application/xls", :xls
```
2.修改`controller`文件
```
class EventsController < ApplicationController
  def export
    today = Time.now.strftime("%Y%m%d")
    file_name = "events_#{today}.xls"
    respond_to do |format|
      format.xls { headers["Content-Disposition"] = "attachment; filename=\"#{file_name}\""}
      format.all { head :not_acceptable }
    end
  end
end
```
测试 [http://localhost:3000/events/export.xls](http://localhost:3000/events/export.xls)
