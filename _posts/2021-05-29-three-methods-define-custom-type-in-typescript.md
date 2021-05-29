---
layout: post
title: TypeScript里创建类型的三种方法
date: 2021-05-29 19:29 +0800
---

在 typescript  里我们可以使用关键字 type, interface 及 class 创建类型。比如我们想定义类型 Person。
## type 版本
#### 定义
``` js
type Person = {
    name: string,
    age: number
}
```
#### 使用方法
``` js
const person: Person = {
    name: 'andrew',
    age: 34
}
```
## interface 版本
#### 定义
``` js
interface Person {
    name: string;
    age: number;
}
```
#### 使用方法
``` js
const person: Person = {
    name: 'andrew',
    age: 34
}
```
## class 版本
#### 定义
``` js
class Person {
    name: string;
    age: number;
}
```
#### 使用方法
``` js
const person = new Person()
person.name = 'andrew'
person.age = 34
```

## 三种方法如何取舍？

1. 如果不需要实例化对象，则选用 type 或者 interface，否则就选用 class；
2. 如果需要使用 instanceof 判断对象的类型，则使用 class；
3. 如果只是为了保证变量的类型安全，则使用 type 及 interface 就够了，type 及 Interface 不会被转译成 JavaSript，也减小了编译之后的 JavaScript 文件大小；
4. 默认使用 interface 的方法，即能用 interface 就使用 interface，否则使用 type。