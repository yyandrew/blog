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
原因其实很简单，你也许可能猜出来。`count`会产生一次`count sql`。而`size`获得数组的大小。

我们再看看它们的源码：
```
menu.sub_menu_items.method(:size).source.display
```
输出
```
def size
  loaded? ? @records.length : count(:all)
end
```
看起来调用了`length`方法，我们继续看看`length`是如何实现的
```
menu.sub_menu_items.method(:length).source.display

```
输出
```
    delegate :to_xml, :encode_with, :length, :each, :join,
             :[], :&, :|, :+, :-, :sample, :reverse, :rotate, :compact, :in_groups, :in_groups_of,
             :to_sentence, :to_formatted_s, :as_json,
             :shuffle, :split, :slice, :index, :rindex, to: :records
```
可见调用`length`会被转发到`records`这个数组的`length`的调用.

`count`其实上最后调用`build_count_subquery`，去查询数据库。它的源码是
```
def build_count_subquery(relation, column_name, distinct)
  if column_name == :all
    column_alias = Arel.star
    relation.select_values = [ Arel.sql(FinderMethods::ONE_AS_ONE) ] unless distinct
  else
    column_alias = Arel.sql("count_column")
    relation.select_values = [ aggregate_column(column_name).as(column_alias) ]
  end

  subquery_alias = Arel.sql("subquery_for_count")
  select_value = operation_over_aggregate_column(column_alias, "count", false)

  relation.build_subquery(subquery_alias, select_value)
end
```
