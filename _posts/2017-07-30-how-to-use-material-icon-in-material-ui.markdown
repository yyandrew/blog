---
layout: post
title: "在material-ui中使用material icons"
date: 2017-07-30 22:02:00 +0800
comments: true
categories: "React"
---

---

### 需要用到的资源:

* [material icons](https://material.io/icons/)
* [material-ui](http://www.material-ui.com/)

1, 打开[material icons](https://material.io/icons/), 搜索需要的icon。比如我们现在想要使用**android机器人**的图标，在搜索框里输入**android**。我们会得到3个结果。它们分别在**Action**和**Hardware**类别里。我们想要的是**Action**下面的**android**。我们先记下它们。

2, 在react component中引入AndroidIcon组件

```js
// 注意规则:action/android对应第一步的Action和android，前面的规则一般不变
import AndroidIcon from 'material-ui/svg-icons/action/android';
```

这时时你就可以使用`<AndroidIcon />`

