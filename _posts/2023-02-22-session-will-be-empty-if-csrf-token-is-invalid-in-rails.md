---
layout: post
title: 一次 X-CSRF-Token 引起的Rails 项目 session 为空的问题
date: 2023-02-22 10:16 +0800
---
### 问题描述
在页面上使用 ajax 发送异步请求无法验证用户。猜测是因为cookies非法导致的。

### 尝试解决过程
1. 通过浏览器的开发工具->网络查看 Request Headers 发现 Cookie 是有一起随请求发送给服务端的，于是有了第二步；
2. 通过在 controller 使用 [cookies](https://apidock.com/rails/ActionController/Cookies/cookies) 方法打印当前用户的 cookies 发现用户的 cookies 为空，返回值为`ActionController::RequestForgeryProtection::ProtectionMethods::NullSession::NullCookieJar`。[ NullCookieJar 产生的原因](https://stackoverflow.com/a/36733432)是 `verify_authenticity_token` 方法验证 Cross-Site Request Forgery (CSRF) 失败造成的。

### 解决方案
有两种方案可以解决这个问题

方案一: 在相应的 controller 里添加 `skip_before_action :verify_authenticity_token` 跳过 CSRF 验证即可以解决问题。(本例使用的方案)。

方案二: 由服务端生成一个 X-CSRF-Token 并将其隐藏在页面上，在发送 ajax 请求时就这个 X-CSRF-Token 添加到请求头，随着请求一起发送到服务端。


参考链接：
1. [Difference between request.cookies and cookies in a controller](https://stackoverflow.com/a/45919902)
