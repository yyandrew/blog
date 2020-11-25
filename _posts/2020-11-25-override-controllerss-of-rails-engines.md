---
layout: post
title: 重写Rails项目里的Engine的Controller
date: 2020-11-25 14:54 +0800
category: Rails
---

有时候我们需要修改Rails Engines的Controllers的方法。那么可以使用open class的方式修改它。

比如我们有一个Rails Engine `MyEngine`

```ruby
# Rails Engine里的`app/controllers/myengine/my_controller.rb`
module MyEngine
  class MyController < ApplicationController
    def index
    end
  end
end
```

我们可以使用下面的代码重写`MyController#index`:

```ruby
# 主项目里的`app/controllers/myengine/my_controller.rb`
# 由于Rails只会加载一次Controller
# 其它同名的Controller会被忽略
# 因此必须用`require_dependency`手动加载MyEngine的Controller，这样才实现了重写
require_dependency MyEngine::Engine.config.root.join("app", "controllers", "myengine", "my_controller").to_s

class MyController
  def index
    # Override index action here
  end
end
```

Rails官网有[更复杂的实现](https://guides.rubyonrails.org/v5.0/engines.html#overriding-models-and-controllers)

