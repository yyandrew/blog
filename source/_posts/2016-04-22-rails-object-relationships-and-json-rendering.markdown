---
layout: post
title: "怎么在不添加冗余字段的情况下统计子model的数量"
date: 2016-04-22 12:08:02 +0800
comments: true
categories: "Rails"
---
在做API的时候会有这样的需求，我想知道一个client的product的数量，但是又不想在client中添加一个冗余的字段，这时你可以试试`include`的`methods`这个选项。

假如我们有`client`,`products`两个model,它们的关系是一对多的关系，即一个`client`可以包含多个`product`。如下代码所示：

``` ruby app/models/client.rb
class Client < ActiveRecord::Base
  has_many :products, dependent: :destroy
end
```

``` ruby app/models/product.rb
class Product < ActiveRecord::Base
  belongs_to :client
end
```

要使用methods方法，你必须在`app/models/client.rb`中添加一个方法，用来统计这个`client`有多少个`product`s.

```ruby app/models/client.rb
class Client < ActiveRecord::Base
  belongs_to :client

  def products_count
    Product.where(client_id: self.id)
  end
end
```

准备工作已经做完，现在要考虑怎么使用它。

``` ruby
format.json { render {json: Client.limit(10).as_json(include: :products, methods: :products_count)}}
```

提示: 你的as_json可能会包含多个`include`，这时建议将含有`methods`的`include`放在最前面。要是放在其它的`include`的后面的话可能导致这个`include`不产生作用。
