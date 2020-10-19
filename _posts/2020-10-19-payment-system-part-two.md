---
layout: post
title: 购物车支付系统之二(支付方式验证篇)
catetory: Rails
date: 2020-10-19 17:16 +0800
---
### 支付之前的准备

功能描述：显示购买的商品。处理支付方式(信用卡/其它的第三方支付)

#### 涉及到的表

1. users - 保存用户的邮箱，密码，性别等信息。
2. payment_methods - 保存用户的付款方式信息，付款方式(信用卡或者paypal)，隐藏后的卡号，失效日期，是否可用等

#### 数据库表关系图

![pre-pay-eer-diagram.png](/images/pre-pay-eer-diagram.png)

PS:可以[下载mwb文件](/images/pre-pay-eer-diagram.mwb)，使用MySQL Workbench打开查看各个表的字段。

#### 流程图

![pre-pay-workflow.png](/images/payment-workflow.png)
