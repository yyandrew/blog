---
layout: post
title: 嵌入到iframe里的应用用户验证问题解决方案
date: 2023-02-21 12:52 +0800
---
### 问题描述
最近有个新的需求要将现有的后台管理系统（子级系统）嵌入到 iframe 里(在父级系统)使用。在开发的过程中发现了 POST , PATCH , PUT 这类修改数据的请求无法正常使用，会强制跳转到登录页面，但是 GET 获取数据请求却不会跳转到登录页面。

经过搜索得知是因为安全原因 **cookie 不能跨域名**导致 iframe 里触发的修改数据请求不会自动带上 cookie 随请求一起发送到后端。

### 解决方案

一般有下面三种解决方案
1. 服务端设置 cookie 时，将其设置成 SameSite=None 并且要设置 Secure（本例采用的方法）；
2. 将 iframe 里的应用部署到同一个二级域名下（不适用本例）;
3. 使用 jwt 验证（工作量略大，时间上不允许）;
4. 设置代理（不适用本例）。

这两个值的含义分别是：
* SameSite=None: Cookie 将在所有上下文中发送，即允许跨站发送
* Secure: Cookie仅通过 HTTPS 协议加密发送到服务器。
如果只设置 SameSite=None 但是没有设置 Secure 会出现警告，这类 cookie 会被拒绝。


Rails 项目的设置方法：修改`config/initializers/session_store.rb`，内容如下：

``` ruby
YourApplication::Application.config.session_store :cookie_store, key: '_your_application_session'
```

参考链接：
1. [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie/SameSite](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
