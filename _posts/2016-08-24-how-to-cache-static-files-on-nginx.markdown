---
layout: post
title: "设置nginx缓存网站静态文件"
date: 2016-08-24 07:29:50 +0800
comments: true
categories: "Nginx"
---
### 设置`expires`
`expires`可以设置在在nginx配置文件中的http {}, server {}或者location {}中，但是通常被放在location {}中
``` irc
location ~*  \.(jpg|jpeg|png|gif|ico|css|js)$ {
  expires 365d; # 缓存有效期是一年
}
```
上面的配置是缓存.jpg, .jpeg, .png, .gif, .ico, .css, .js，有效期为一年。

#### `expires`的格式

* off 不使用cache

* 没有＠前缀的时间间隔，表示从浏览器访问的当前时间开始计时。格式可以为[x]y[x]M[x]w[x]d[x]h[x]m[x]s[x]ms，其中y表示年（365天）,M为月（30天），w为周，d为天，h为小时，m为分钟，s为秒，ms为毫秒。如expires 3d就表示第一次访问网站后３天后cache失效

* 以＠前缀的时间点，到达当天时间点后会就会过期。格式为Hh或者Hh:Mm，H取值范围是0－24，M取值范围为0－59。 例如`@14:00`就表示下午２点后cache失效

### 测试
我们可以通过`chromium`的network工具查看上面`365d`是否设置成功

不要忘了先重启一下nginx让修改生效`sudo /etc/init.d/nginx reload`

![nginx-cache-test](/images/nginx-cache-test.png)
