---
layout: post
title: Rails 捕获自定义的异常
---
1. 在项目中新建一个文件`app/controllers/concerns/error_handlable.rb`，内容如下
```ruby
module ErrorHandlable
  extend ActiveSupport::Concern
  # 自定义一个异常
  class MyError < StandardError; end

  included do
    # rescue_from 是 Rails 提供的一个捕获异常的方法
    rescue_from MyError do |e|
      render_error e.message
    end

    def render_error(message)
      render json: { status: 400, message: message}
    end
  end
end
```

2. 在 controller 里的 action 添加下面的逻辑
```ruby
class FooController < ApplicationController
  include ErrorHandlable

  def index
    @bar = Bar.find_by id: params[:id]

    # 如果没有找到记录，就抛出异常
    raise MyError, 'Record not found' unless @bar

    respond_to do |format|
      format.json { render json: { bar: @bar } }
    end
  end
end
```

参考链接：
1. [rescue_from文档](https://apidock.com/rails/ActiveSupport/Rescuable/ClassMethods/rescue_from)
