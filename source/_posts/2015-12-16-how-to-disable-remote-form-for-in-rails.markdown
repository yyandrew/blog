---
layout: post
title: "禁止remote:true属性的form_for提交"
date: 2015-12-16 17:31
comments: true
categories: "Rails"
---
在做form前端验证的时候发现即使对表单onsubmit返回false也不能阻止表单的提交。
form_for代码
```rb
= form_for @user, html: {class: register-form}, remote: true do |f|
  = f.text_field :name
```
错误的JavaScript代码
```js
$(document).on("submit", "register-form", function(){
  return false;
})
```
经过调试发现是form_for的remote属性在搞鬼。原因是表单在submit事件调用之前它的ajax的事件就已经被触发了。
正确的解决方法是在ajax的beforesend事件之前返回false
正确的JavaScript代码
```js
$(document).on("ajax:beforeSend", "register-form", function(){
  return false;
})
```
