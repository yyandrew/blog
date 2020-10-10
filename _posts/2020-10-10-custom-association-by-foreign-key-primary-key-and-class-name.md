---
layout: post
title: 使用foreign_key, primary_key及class_name设置关联关系
date: 2020-10-10 16:41 +0800
category: Rails
---

首先看看我们用到的三个选项：

1. foreign_key - 设定要使用的外键名。按照约定ActiveRecord会使用模型名加`_id`这种格式去猜foreign_key，比如`UserServiceAccess`里有`has_one`关系，ActiveRecord会使用`user_service_access_id`为外键名。
2. primary_key -  设定主键值。按照约定ActiveRecord会使用`user_service_access.id`为主键值。
3. class_name - 如果另一个模型无法从关联的名称获取，可以用它来设定模型。比如下面的例子设定了`has_one current_user_service_access_offer`,ActiveRecord会使用`CurrentUserServiceAccessOffer`为模型名。

涉及到的模型

* UserServiceAccess

```ruby
class UserServiceAccess < ApplicationRecord
  has_one :current_user_service_access_offer, foreign_key: :id, primary_key: :current_offer_id, class_name: 'UserServiceAccessOffer'
end
# == Schema Information
#
# Table name: user_service_access
#
#  id                    :integer          not null, primary key
#  current_offer_id      :integer
```

* UserServiceAccessOffer

```ruby
class UserServiceAccessOffer < ApplicationRecord
end
# == Schema Information
#
# Table name: user_service_access_offer
#
#  id                                   :integer          not null, primary key
```

测试

```shell 
[34] pry(main)> usa = UserServiceAccess.first  
[36] pry(main)> usa.current_offer_id                                                     
=> 144   
[37] pry(main)> usa.current_user_service_access_offer
  UserServiceAccessOffer Load (0.4ms)  SELECT `user_service_access_offer`.* FROM `user_service_access_offer` WHERE `user_service_access_offer`.`deactivated_at` IS NULL AND `user_service_access_offer`.`id` = 144 LIMIT 1 
```

从上面的测试可以看到`UserServiceAccess`的`current_offer_id`是`144`，查找它的`current_user_service_access_offer`产生的SQL也正确的使用的id`user_service_access_offer.id = 144`这个条件。