---
layout: post
title: "手机版safari的一些小问题"
date: 2015-11-17 19:31
comments: true
categories: "CSS"
---
1.当你想输入一些内容而tap一个输入框时，这个输入框它会自动被放大,处理方法就是设置设置输入框的文字大小为16px
```css
input, textarea {
  font-size: 16px;
}
```
2.在手机版safari中当你tap一个link或者绑定click事件的元素时候，这个元素闪烁([被高亮](https://developer.apple.com/library/safari/documentation/AppleApplications/Reference/SafariWebContent/AdjustingtheTextSize/AdjustingtheTextSize.html))一下，可以通过下面方法消除它的闪烁
```css
.target-elem {
  -webkit-tap-highlight-color: rgba(0,0,0,0);
}
```
