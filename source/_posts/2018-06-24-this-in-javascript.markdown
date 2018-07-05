---
layout: post
title: "JavaScript中的this详解"
date: 2018-06-24 09:59:40 +0800
comments: true
categories: "JavaScript"
---

## 全局this

在全局环境中this等价于`window`对象
```
console.log(this == window) // 输出:true
```
## 存在于函数中的this

函数中的`this`相当于`window`

``` javascript
function this_in_func() {
  console.log(this == window)
}

this_in_func() // 输出:true.注意如果运行使用了`strict`模式，这时this为undefined
```

``` javascript
function Hero(heroName, realName) {
  this.heroName = heroName
  this.realName = realName
}
let supername = new Hero('Superman', 'Clark Kent')
console.log(window.heroName == supername.heroName) // 输出:true
```

## 方法中的this

当调用一个函数的时候，`this`相当于当前的调用者。比如下面例子中`dialogue`方法中的`this`就是`hero`

``` javascript
const hero = {
  heroName: 'Batman',
  dialogue() {
    console.log(`I am ${this.heroName}`)
  }
}
hero.dialogue() //输出:I am Batman
```

## call()和apply()中的this

在`call()`和`apply()`中，`this`相当于接收者对象
``` javascript
function dialogue() {
  console.log(`I am ${this.heroName}`)
}
const hero = {
  heroName: 'Batman'
}
dialogue.call(hero) // 输出:I am Batman
dialogue.apply(hero) // 输出:I am Batman
```

## bind()和this

当一个方法作为一个回调函数参数传递给另外一下函数时，this相当于window，而不是接收者对象。但是bind()方法可以避免产生这样的问题。

比如下面的例子，`this`被赋值成`hero`对象

``` javascript
const hero = {
  heroName: 'Batman',
  dialogue() {
    console.log(`I am ${this.heroName}`)
  }
}
setTimeOut(hero.dialogue.bind(hero), 1000); // 输出:I am Batman
```

## 箭头函数中的this

箭头函数中的`this`取父执行上下文中的`this`的值。比如下面的例子中输出的并不是hero对象的heroName值(Batman)，而是hero上下文中的heroName的值(Superman)。

``` javascript
heroName = 'Superman';
const hero = {
  heroName: 'Batman',
  dialogue: () => {
    console.log(this.heroName)
  }
}
hero.dialogue() // 输出:Supername
```

## 类中的this

在类中`this`指向这个类的实例。

``` javascript

class Hero {
  constructor(heroName) {
    this.heroName = heroName
  }

  dialogue() {
    console.log(`I am ${this.heroName}`)
  }
}

let hero = new Hero('Batman')
hero.dialogue() // 输出:I am Batman
```

但是当我们把`hero.dialogue`赋值给一个变量时`this`会变成`undefined`，因为调用`dialogue`方法的调用者发生的改变。
```
let say = hero.dialogue
say() // 会报错:Uncaught TypeError: Cannot read property 'heroName' of undefined
```

解决方法就是手动`bind`将`dialogue()`和对象`hero`联系起来
``` javascript
let say = hero.dialogue.bind(hero);
say() // 输出：I am Batman
```