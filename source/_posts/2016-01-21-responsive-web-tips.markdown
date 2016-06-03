---
layout: post
title: "响应式网站笔记"
date: 2016-01-21 11:04
comments: true
categories: "CSS"
---
1,利用css3的flex使元素垂直居中[stackoverflow]( http://stackoverflow.com/a/22218694 )
```css
.targetDiv {
  display: flex;
  align-items: center;
}
```
2.去掉IE10+中input[type='text']的x按钮
```css
input[type="tex"]::-ms-clear {
  display: none;
}
```
3.控制input的placeholder的样式

``` css
::-webkit-input-placeholder {
   color: red;
}

:-moz-placeholder { /* Firefox 18- */
   color: red;
}

::-moz-placeholder {  /* Firefox 19+ */
   color: red;
}

:-ms-input-placeholder {
   color: red;
}
```

4.将4个block等宽，间距相等一行排列
  ![blocks](/images/four-blocks.png)

``` html html
<div class="item-blocks">
  <div class="block">
    <div class="block1">This is block1</div>
  </div>
  <div class="block">
    <div class="block2">This is block2</div>
  </div>
  <div class="block">
    <div class="block3">This is block3</div>
  </div>
  <div class="block">
    <div class="block4">This is block4</div>
  </div>
</div>
```

``` scss scss
.item-blocks {
  margin: 0 -5px;
  .block {
    width: 25%;
    float: left;
    .block1, .block2, .block3, .block4 {
      background-color: #F6F6F6;
      height: 125px;
      margin: 0 5px;
    }
  }
}

```
