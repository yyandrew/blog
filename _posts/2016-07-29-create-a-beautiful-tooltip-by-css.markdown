---
layout: post
title: "使用纯CSS创建一个圆角tooltip效果"
date: 2016-07-29 10:29:52 +0800
comments: true
categories: "CSS"
---
主要用到的CSS的属性: `position`, `border`

``` HTML HTML代码
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.0/jquery.min.js"></script>
<div class="circle">
  <div class="tooltip">
    <p class="tip-con">tip content1</div>
    <p class="arrow-down"></p>
  </div>
</div>
<div class="circle">
  <div class="tooltip">
    <p class="tip-con">tip content2</div>
    <p class="arrow-down"></p>
  </div>
</div>
```

```CSS CSS代码
.circle {
  border-radius: 50%;
  float: left;
  text-align: center;
  line-height: 43px;
  margin: 100px 5px 0 0;
  cursor: pointer;
  position: relative;
  background-color: #cccccc;
  height: 40px;
  width: 40px;
}
.circle:hover .tooltip {
  display: block;
}
.circle:hover .arrow-down {
  display: block;
}
.tooltip {
  display: none;
  height: 38px;
  line-height: 39px;
  text-align: center;
  position: absolute;
  left: 0;
  background-color: #cccccc;
  font-family: "PTSans";
  font-weight: normal;
  font-size: 14px;
  top: -50px;
  border-radius: 4px;
}
p.tip-con {
  margin: 0 20px;
  white-space: nowrap;
  color: white;
}
p.arrow-down {
  display: none;
  width: 0;
  height: 0;
  margin: 0;
  position: absolute;
  top: -12px;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-top: 5px solid #cccccc;
}
```

``` JAVASCRIPT JavaScript代码
$(".circle").on("mouseover", function(){
  $tooltip = $(this).find(".tooltip");
  widthOfTooltip = $tooltip.width();
  widthOfCircle = $(this).width();
  $tooltip.css("left", -(widthOfTooltip - widthOfCircle)/2);
  console.log(widthOfTooltip/2-5);
  $(this).find(".arrow-down").css("left", widthOfCircle/2-5);
})
```
效果[jsfiddle](https://jsfiddle.net/vn2xtxsp/)
