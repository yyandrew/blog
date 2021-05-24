---
layout: post
title: 使用react，typescript开发electron应用
date: 2021-05-24 23:10 +0800
---

#### 创建一个简单的 electron 项目
1. 初始化一个项目
    ``` shell
    yarn init -y
    yarn add -D electron
    ```
2. 创建 electron 入口文件

    ```javascript
    // src/electron.js
    const { app, BrowserWindow } = require('electron')

    function createWindow() {
      let win = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
          nodeIntegration: true,
        },
      })
      win.loadFile('./index.html')
    }

    app.on('ready', createWindow)
    ```
3. 创建 electron 界面入口文件

    ``` html
    <!-- // src/index.html -->
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width">
        <title>Hello world</title>
      </head>
      <body>
        <div id="app">
          <h1>Hello world</h1>
        </div>
      </body>
    </html>
    ```
4. 在 package.json 添加一个命令
    ```json
    "scripts": {
      "start": "electron src/electron.js"
    }
    ```
运行 `yarn start` 可以查看效果。

#### 添加 typescript
1. 安装 typescript 依赖。`yarn add -D typescript`；
2. 添加 typescript 配置文件，`touch tsconfig.json`；
3. 修改 package.json，添加一个命令将 typescript 文件编译成 js 文件；
    ```json
    "scripts": {
        "build": "tsc src/electron.ts"
    }
    ```
4. 重命名 `electron.js` 为 `electron.ts`。

#### 添加 webpack
1. 安装依赖。`yarn add -D webpack webpack-cli ts-loader`
2. 添加 webpack 配置文件。
    ``` javascript
    // webpack.config.js
    module.exports = {
      mode: 'development',
      entry: './src/electron.ts',
      target: 'electron-main',
      module: {
        rules: [
          {
            test: /\.ts$/,
            include: /src/,
            use: [{ loader: 'ts-loader' }],
          },
        ],
      },
      output: {
        path: __dirname + '/src',
        filename: 'electron.js',
      },
    }
    ```
配置项目的各个含义为：
* `mode: development` 开发环境；
* ` entry: './src/electron.ts'` 入口文件地址；
* `target: 'electron-main'` webpack 自动检测打包 electron 主进程代码文件；
* ` test: /\.ts$/` 规则会作用于 `.ts` 结尾的文件；
* `include: /src/` `src` 文件夹里的文件会被使用规则；
* `use: [{ loader: 'ts-loader' }]` 对于匹配成功的文件使用什么插件进行打包；
* `path: __dirname + '/src'` 打包好的文件的输出目录；
* `filename: 'electron.js'` 打包文件的文件名。
3. 修改 build 命令
    ```json
    // package.json
      "scripts": {
        "start": "electron src/electron.js",
        "build": "webpack --config ./webpack.config.js"
      }
    ```

#### 添加 react
1. 添加依赖。`yarn add -D react react-dom @types/react @types/react-dom`
2. 添加 React 入口文件。
    ```javascript
    // src/react.tsx
    import * as React from 'react'
    import * as ReactDOM from 'react-dom'

    export const Index = () => <h2>Hello React!</h2>

    ReactDOM.render(<Index />, document.querySelector('#app'))
    ```
3. 添加配置使得 typescript 能够识别编译 tsx 文件。
    ```json
    // tsconfig.json
    {
      "compilerOptions": {
        "jsx": "react"
      }
    }
    ```
4. 添加新的入口文件。首先添加新的依赖 `yarn add -D html-webpack-plugin`。
    ```javascript
    // webpack.config.js
    const HtmlWebpackPlugin = require('html-webpack-plugin')

    module.exports = [
      ...
        {
        mode: 'development',
        entry: './src/react.tsx',
        target: 'electron-renderer',
        devtool: 'source-map',
        module: {
          rules: [
            {
              test: /\.ts(x?)$/,
              include: /src/,
              use: [{ loader: 'ts-loader' }],
            },
          ],
        },
        output: {
          path: __dirname + '/dist',
          filename: 'react.js',
        },
        plugins: [
          new HtmlWebpackPlugin({
            template: './src/index.html',
          }),
        ],
      },
    ]
    ```
`target: 'electron-renderer'` webpack 自动检测打包 electron 渲染程代码文件；
5. 修改 electron.js 的输出目录为 `./dist/`
    ```javascript
    // webpack.config.js
    ...
      output: {
        path: __dirname + '/dist',
        filename: 'electron.js'
      }
    ```
6. 修改 start 命令
    ```yaml
    "scripts": {
      ...
      "start": "yarn build && electron ./dist/electron.js"
    }
    ```

运行 `yarn start` 命令，如果能够看到如下的界面，说明环境搭建成功了。

![final-electron-app-by-react-and-typescript.png](/images/final-electron-app-by-react-and-typescript.png)
