---
layout: post
title: "利用size避免N+1查询"
date: 2015-09-11 11:22
comments: true
categories: "Rails"
---
比如说我们有两个model.它们分别是MenuItem和SubMenuItem
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
end
```
```ruby
# app/views/admin/menu_items/index.html.haml
- @menu_items.each do |menu|
  %tr
    %td= menu.position
    %td= menu.sub_menu_items.count # << 查询menu_item有多少个sub_menu_item

```
观察console后发现log有出现N+1查询的问题
```
MenuItem Load (0.9ms)  SELECT "menu_items".* FROM "menu_items"   ORDER BY menu_items.position ASC
SubMenuItem Load (1.0ms)  SELECT "sub_menu_items".* FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" IN (1, 2, 3, 13, 4, 5, 6, 7, 8, 9, 10, 11, 12)  ORDER BY sub_menu_items.position ASC
 (0.6ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 1]]
 (0.6ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 2]]
 (1.2ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 3]]
 (0.6ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 13]]
 (1.2ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 4]]
 (0.9ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 5]]
 (9.3ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 6]]
 (0.9ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 7]]
 (0.6ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 8]]
 (0.5ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 9]]
 (0.8ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 10]]
 (0.5ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 11]]
 (0.6ms)  SELECT COUNT(*) FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" = $1  [["menu_item_id", 12]]
```
通过分析发现只需要把**app/views/admin/menu_items/index.html.haml**文件中的`%td= menu.sub_menu_items.count`改成`%td= menu.sub_menu_items.size`后就可以避免类似的N+1查询问题，修改之后的log会干净许多
```
MenuItem Load (2.9ms)  SELECT "menu_items".* FROM "menu_items"   ORDER BY menu_items.position ASC
SubMenuItem Load (2.8ms)  SELECT "sub_menu_items".* FROM "sub_menu_items"  WHERE "sub_menu_items"."menu_item_id" IN (1, 2, 3, 13, 4, 5, 6, 7, 8, 9, 10, 11, 12)  ORDER BY sub_menu_items.position ASC
```

