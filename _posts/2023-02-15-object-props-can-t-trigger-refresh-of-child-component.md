---
layout: post
title: Vue里父组件的对象修改无法触发子组件的更新问题解决方案
date: 2023-02-15 17:48 +0800
---

### 解决方案
方案一：在声明`errors`的时候将属性也一并声明好，比如
``` javascript
data() {
  return {
    errors: {
      foo: null,
      bar: null,
    }
  }
}
```

{:start=>2}
方案二：使用`Vue.set`(它的别名为`vm.$set`)在动态添加属性的同时将属性设置成响应式的。官方文档是这样描述`Vue.set`的
> 对于已经创建的实例，Vue 不允许动态添加根级别的响应式 property。但是，可以使用 Vue.set(object, propertyName, value) 方法向嵌套对象添加响应式 property
于是有了下面的代码：
``` javascript
// 添加属性
this.$set(this.errors, 'foo', '错误一')
this.$set(this.errors, 'bar', '错误一')

// 删除对象属性
// this.$delete(this.errors, 'foo')
// this.$delete(this.errors, 'bar')
```
