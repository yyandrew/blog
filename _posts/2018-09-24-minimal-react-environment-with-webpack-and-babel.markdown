---
layout: post
title: "使用webpack和babel配置ReactJS开发环境"
date: 2018-09-24 14:24:05 +0800
comments: true
categories: "React.JS"
---

### [什么是ReactJS](https://reactjs.org.cn/)

它是由Facebook开源的用来构建用户界面的JavaScript库。它采用虚拟DOM机制向DOM填充数据。更新单独的虚拟DOM组件比更新整个DOM树要快。

### [什么是Webpack](https://www.webpackjs.com/)

它是一个模块打包工具。它的主要目标是将JavaScript文件打包在一起之后在浏览器中使用。同时它还可以用来转换(transform)。

### [什么是Babel](https://www.babeljs.cn/)

语法转换工具。将最新的JavaScript语法转换成浏览器支持的语法，无需等到浏览器支持这些最新的语法。

### 创建一个ReactJS项目
一、 创建一个新的项目目录，及基本的文件。
    
``` sh
mkdir reactjs-demo
cd reactjs-demo
npm init -y
mkdir dist
cd dist
```
二、 新建一个`dist/index.html`文件
``` html
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ReactJS示例</title>
  </head>

  <body>
    <div id="app"></div>
    <script src="/bundle.js"></script>
  </body>
</html>
```

其中`bundle.js`是由`Webpack`产生的一个打包文件。

此时项目的目录结构为:

``` sh

├── dist
│   └── index.html
└── package.json
```

### 配置Webpack
一、安装Webpack
``` sh
npm install --save-dev webpack webpack-dev-server webpack-cli
```
二、修改`package.json`
``` JavaScript
...
"scripts": {
  "start": "webpack-dev-server --config ./webpack.config.js --mode development",
  ...
},
...
```
三、新建`webpack.config.js`文件
``` JavaScript
module.exports = {
  entry: './src/index.js',
  output: {
    path: __dirname + '/dist',
    publicPath: '/',
    filename: 'bundle.js'
  },
  devServer: {
    contentBase: './dist'
  }
}
```

其中`entry`是入口起点，它指示webpack应该使用哪个模块，来作为构建其内部依赖图的开始。`output`告诉webpack在哪里输出它所创建的打包文件。`devServer`指定`dist`文件夹为资源文件夹。

四、创建`src/index.js`文件
``` sh
mkdir src
cd src
touch index.js
```
`src/index.js`的内容
```
console.log('ReactJS示例');
```
此时的目录结构为：
``` sh
├── dist
│   └── index.html
├── node_modules
├── package.json
├── package-lock.json
├── src
│   └── index.js
└── webpack.config.js
```

五、启动项目
    ``` sh
    npm start
    ```

六、在浏览器中打开[http://localhost:8080](http://localhost:8080)，不出意外此时在浏览器的console里会打印出`ReactJS示例`。

### 配置Babel

一、安装依赖包
``` sh
npm install --save-dev @babel/core @babel/preset-env babel-loader @babel/preset-react
```
二、修改文件`webpack.config.js`内容
``` JavaScript
...
entry: './src/index.js',
module: {
  rules: [
    {
      test: /\.(js|jsx)$/,
      exclude: /node_modules/,
      use: ['babel-loader']
    }
  ]
},
resolve: {
  extensions: ['*', '.js', '.jsx']
},
...
```
三、在项目根目录下创建新文件`.babelrc`
``` sh
touch .babelrc
```

`.babelrc`文件内容如下：

``` JavaScript
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ]
}
```

### 使用ReactJS

一、在项目根目录下安装如下依赖
``` sh
npm install --save react react-dom
```
二、修改`webpack.config.js`配置自动刷新浏览器
``` JavaScript
var webpack = require('webpack')
...
plugins: [
  new webpack.HotModuleReplacementPlugin()
],
devServer: {
  ...
  hot: true
}
...
```
三、修改文件`src/index.js`内容使用ReactJS
``` JavaScript
import React from 'react';
import ReactDOM from 'react-dom';

const title = 'ReactJS示例';

ReactDOM.render(
  <div>{title}</div>,
  document.getElementById('app')
)
```

[项目完整代码](https://github.com/yyandrew/minimal_react_environment_by_webpack_and_babel)
