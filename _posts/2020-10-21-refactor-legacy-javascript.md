---
layout: post
title: 使用babel全家桶重构JavaScript遗留代码
date: 2020-10-22 17:11 +0800
categories: "JavaScript"
---

### 什么是遗留代码

任何已经存在的、难以维护或者难以扩展的代码都是遗留代码。

比如下面代码

```javascript 
var Person = function () {
  this.name = "andrew";
  this.age = 33;
  this.title = 'web developer';
};
Person.prototype.sayHello = function () {
  console.log("Hello: ", this.name);
};
Person.prototype.sayAge = function () {
  console.log("Age: ", this.age);
};
Person.prototype.sayTitle = function () {
  console.log("Title: ", this.title);
};
var andrew = new Person();
andrew.sayHello();
andrew.sayAge();
andrew.sayTitle();
```

上面的代码包含三个的原型方法，现在看起来好像没有太大的问题，随着时间的流逝这三个函数包含的代码量可能会增加，从而会导致这个文件会越来越大。

因此我们希望把这些原型方法移到单独的文件中，使用ES6的模块方式管理它们。比如下面的代码是我们想看到的。

```javascript 
import sayTitle from "./sayTitle";
import sayAge from "./sayAge";
import sayHello from "./sayHello";

var Person = function () {
  this.name = "andrew";
  this.age = 33;
  this.title = 'web developer';
};

Person.prototype.sayHello = sayHello;
Person.prototype.sayAge = sayAge;
Person.prototype.sayTitle = sayTitle;
var andrew = new Person();
andrew.sayHello();
andrew.sayAge();
andrew.sayTitle();
```

`sayXXX`声明被移到各自的文件里，之前匿名函数被方法名替换，现在代码看起来简洁了许多。

下面我们使用babel全家桶开始重构代码。主要的流程是:读取遗留代码的源码->使用`@babel/parser`将源码转换成AST->从AST提取匿名函数->将匿名函数以写入文件并使用ES6的export导出->将文件import到遗留代码->修改AST将匿名函数替换成方法名。

下面的代码实现了整个重构过程。

```javascript
const fs = require("fs");
const { resolve } = require("path");
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const generator = require("@babel/generator").default;
const t = require("@babel/types");

const SOURCE_CODE = resolve("./index.js");

const TARGET_FOLDER = resolve("outputs");

// 读入源代码
const code = fs.readFileSync(`${SOURCE_CODE}`, "utf-8");
// 将源代码转换成AST
const ast = parser.parse(code);

let writeToFile = function (funcName, funcCode) {
  fs.writeFileSync(`${TARGET_FOLDER}/${funcName}.js`, funcCode);
};

let createImportDeclaration = function (funcName) {
  return t.importDeclaration(
    [t.importDefaultSpecifier(t.identifier(funcName))],
    t.stringLiteral(`./${funcName}`)
  );
};

traverse(ast, {
  AssignmentExpression({ node }) {
    const { left, right } = node;
    // 如果当前节点的左右子节点为成员表达式节点及函数表达式节点
    // 则需要提取函数
    if (
      left.type === "MemberExpression" &&
      right.type === "FunctionExpression"
    ) {
      // 获取表达式节点及属性名称
      const { object, property } = left;
      // 检查节点的属性名称是否为prototype
      if (object.property.name === "prototype") {
        const funcName = property.name;
        // 将AST重新生成源码
        const { code: funcCode } = generator(right);
        const replaceNode = t.identifier(funcName);
        // 修改右子树，用函数名替换函数声明
        node.right = replaceNode;
        // 将函数声明写入单独的文件
        writeToFile(funcName, `export default ${funcCode}`);
        // 将模块引入到旧的遗留代码中
        ast.program.body.unshift(createImportDeclaration(funcName));
      }
    }
  },
});
// 将AST重新生成源码，并写入到遗留代码。
writeToFile('refacted_code', generator(ast).code);
```

参考链接

1. AST查看工具http://astexplorer.net/
2. AST文档 https://developer.mozilla.org/zh-CN/docs/Mozilla/Projects/SpiderMonkey/Parser_API
3. 完整救命代码 https://github.com/yyandrew/use-babel-refactor-legacy-code