---
layout: post
title: "使DOM元素保持在最底部的两种方法"
date: 2015-09-22 16:42
comments: true
categories: "CSS"
---
1.如果元素的父元素是body，那么我们可以通过以下简单的方法置底
```css
.target-elem {
  position: fixed;
  bottom: 0;
}
```
2,如果想将子元素固定在父元素的底部，我们可以利用下面方法
```css
.parent-elem {
  position: relative;
  .child-elem {
    position: absolute;
    bottom: 0;
  }
}
```
