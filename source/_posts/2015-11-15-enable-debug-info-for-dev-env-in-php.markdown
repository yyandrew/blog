---
layout: post
title: "开启php的调试模式"
date: 2015-11-15 16:42
comments: true
categories: "PHP"
---
在安装php的应用的时候会遇到问题，但是没有错误提示，之所以会看不到是因为php.ini将这些信息隐藏了。我们可以通过设置php.ini让这些错误信息显示出来。
```php
error_reporting = E_ALL
display_errors = On
```
或者也可以将这两行代码放在.htaccess文件中：
```php
php_flag display_errors on
php_value error_reporting 999999999
```
