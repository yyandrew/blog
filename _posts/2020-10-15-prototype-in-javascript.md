---
layout: post
title: JavaScript里的原型链
category: JavaScript
---

### 定义

早期的JavaScript没有class，JS为了实现对象，使用了比较曲折的方法。 所以原型为其他对象提供共享属性。

```javascript
// ES5方式实现类
function Child() {
 this.name= "Trisha";
}
Child.prototype.sayHello = function() {
 console.log('Hello', this.name)
}
let c = new Child();
c.sayHello()
```

### 对象的隐式引用

所有对象都有一个`__proto__`属性。它是就是我们标题所说的隐式引用。也被称为对象的原型。所谓隐式，是指不是由开发者创建/操作。

### 实例方法查找用__proto__

当你访问一个对象上没有的属性时，对象会去`__proto__`查找，`__proto__`就等于父类的prototype。比如调用`new Foo().bar`，当你访问的方法在Foo.prototype也找不到时，就继续在Object.prototype里找。如果在Object.prototype也找不到就真的找不到了。

`foo.__proto__`指向了`Foo.prototype`。`Foo.prototype.__proto__`指向Object.prototype。

### constructor

我们所说的constructor一般时指原型链上的constructor，即`Foo.prototype.constructor`。它指向函数Foo本身。

修改prototype.constructor只会修改它的指针，不会修改原来的constructor。

### 静态方法

1. 定义

```
Foo.bar = function() {
}
```

2. 调用

```
Foo.bar()
```

静态方法和实例方法的区别在于：静态方法里this拿不到实例。

### 继承

使用原型链不能继承父类的constructor

```javascript
function Parent() {
 this.parentName = "Andrew";
}
function Child() {}
Child.prototype.__proto__ = Parent.prototype
const firstChild = new Child();
console.log(firstChild.parentName); // undefined
```

正确的方法是使用

```javascript 
function Parent() {
 this.parentName = "Andrew";
}
function Child() {}
Child.prototype.__proto__= new Parent();
const firstChild = new Child();
console.log(firstChild.parentName); // Andrew
```

示例图：

![prototype-in-javascript.png](/images/prototype-in-javascript.png)
