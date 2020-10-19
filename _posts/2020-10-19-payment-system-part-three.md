---
layout: post
title: 购物车支付系统之三(付款篇)
category: Rails
date: 2020-10-19 17:20 +0800
---
### 付款流程

功能描述：为购物车的商品付款。

#### 涉及到的表

1. users - 保存用户的邮箱，密码，性别等信息。
2. orders - 保存付款时间，总价，发票(可能是一个第三方支付的账单ID)，类型(等信息。
3. payment_methods - 保存用户的付款方式信息，付款方式(信用卡或者paypal)，隐藏后的卡号，失效日期，是否可用
4. payments - 保存交易号(可能是第三方提供的唯一ID)，支付状态，支付金额，货币类型，订单ID等
5. order_items - 保存购买商品名称，商品数量，商品价格，所属订单，状态等
6. coupons - 保存优惠码，优惠码类型(百分比折扣，或者折扣金额)，起效时间，失效时间，使用次数限制，关联的product,offer及order_item等。

#### 数据库表关系图

![payment-eer-diagram.png](/images/payment-eer-diagram.png)

PS:可以[下载mwb文件](/images/payment-eer-diagram.mwb)，使用MySQL Workbench打开查看各个表的字段。

#### 流程图

![payment-workflow.png](/images/payment-workflow.png)

