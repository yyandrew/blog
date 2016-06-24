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

4.在tooltip和pointer之间划一条虚线

``` javascript
tooltip: {
  useHTML: true,
  borderRadius: 5,
  formatter: function() {
    return '<div class="tooltip-of-chart">' + this.y + '</div>' + this.x;
  },
  positioner: function(labelWidth, labelHeight, point) {
    return {
      x: point.plotX + barChart.plotLeft - 80,
      y: point.plotY + barChart.plotTop - 182
    };
  },
  borderColor: "#979797"
}
plotOptions: {
  series: {
    point: {
      events: {
        mouseOver: function() {
          var xPosition, yPosition;
          var barChart = this.chart
          xPosition = this.plotX + barChart.plotLeft + 1;
          yPosition = this.plotY + barChart.plotTop;
          barChart.dotLine = barChart.renderer.path(['M', xPosition, yPosition - 4, 'L', xPosition, yPosition - 71]).attr({
            "stroke-width": 3,
            stroke: "#CCCCCC",
            dashstyle: "ShortDash"
          }).add();
          return barChart.dotLine.show();
        }
      }
    },
    events: {
      mouseOut: function() {
        var barChart = this.chart
        if (barChart.dotLine) {
          return barChart.dotLine.element.remove();
        }
      }
    }
  }
}
```

5.保持y轴始终在图表的底部

``` javascript
yAxis: {
  minRange: 0.1
}
# 如果是line图表还要添加下面代码
plotOptions: {
  line: {
    softThreshold: false
  }
}
```
