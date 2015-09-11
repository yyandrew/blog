---
layout: post
title: "怎么通过预加载scope避免 N+1查询"
date: 2015-09-13 14:17
comments: true
categories: "Rails"
---
严格来说不是通过预加载scope，而是通过把scope转换成associations。比如有两个model: MenuItem和SubMenuItem.
```ruby
# app/models/menu_item.rb
class MenuItem < ActiveRecord::Base
  has_many :sub_menu_item
end
```
```ruby
# app/models/sub_menu_item.rb
class SubMenuItem < ActiveRecord::Base
  belongs :menu_item
  scope :published, -> {where(status: 1)}
end
```
```ruby
# app/views/admin/menu_items/index.html.haml
- @menu_items.each do |menu|
  %tr
    %td= menu.position
    %td= menu.sub_menu_items.published.count # << 查询menu_item有多少个已经发布了的sub_menu_item

```
我们只需要修改一下两个model
```ruby
# app/models/menu_item.rb
class MenuItem < ActiveRecord::Base
  has_many :sub_menu_item
  has_many :published_sub_menu_items, -> {where(status: 1)}, clas_name: SubMenuItem
end
```
```ruby
# app/models/sub_menu_item.rb
class SubMenuItem < ActiveRecord::Base
  belongs :menu_item
end
```
然后就可以使用includes`MenuItem.includes(:published_sub_menu_items)`。
相应的**app/views/admin/menu_items/index.html.haml**也要修改，将`%td= menu.sub_menu_items.published.count`修改成成`%td= menu.published_sub_menu_items.count`
