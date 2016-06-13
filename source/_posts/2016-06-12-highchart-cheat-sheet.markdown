---
layout: post
title: "highchart小抄"
date: 2016-06-12 17:51:52 +0800
comments: true
categories: "Javascript"
---

1.设置y轴只显示整数

```javascript
yAxis: {
  allowDecimals: false
}
```

2.去掉导出按钮和水印

``` javascript
exporting: {
  enabled: false #去掉导出按钮
},
credits: {
  enabled: false #去掉'Highcharts.com'水印
}
```

3.设置tooltip超过图表部分的内容总是显示

``` javascript
tooltip: {
  useHTML: true,
  formatter: function() {
    return '<div class="tooltip-of-chart">'+this.y + '</div>Total Companies on ' + Highcharts.dateFormat("%b %d, %Y", new Date(this.x))
  }
}
```

``` CSS
.highcharts-container { overflow: visible !important;  }
.highcharts-container svg {
    overflow: visible !important;
}
```
