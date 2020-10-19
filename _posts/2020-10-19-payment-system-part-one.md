---
layout: post
title: 购物车支付系统之一(创建订单)
category: Rails
date: 2020-10-19 17:09 +0800
---
### 创建订单
功能描述： 添加商品到购物车。

#### 涉及到的表

1. users - 保存用户的邮箱，密码，性别等信息。
2. orders - 保存付款时间，总价，发票(可能是一个第三方支付的账单ID)，类型(等信息。
3. offers - 出售商品。保存商品名称，单价，税率，含税价格，货币单元，是否在售卖等信息。
4. order_items - 保存购买商品名称，商品数量，商品价格，所属订单，状态等
5. products - 在售产品。保存产品名称，单价，税率，含税价，库存量，是否在售卖等信息。

#### 数据库表关系图

![create-new-order-eer-diagram.png](/images/create-new-order-eer-diagram.png)

PS:可以[下载mwb文件](/images/create-new-order-eer-diagram.mwb)，使用MySQL Workbench打开查看各个表的字段。

#### 流程图

![create-new-order-workflow.png](/images/create-new-order-workflow.png)
