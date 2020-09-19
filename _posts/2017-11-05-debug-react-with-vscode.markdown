---
layout: post
title: "VSCode调试react项目"
date: 2017-11-05 10:16:21 +0800
comments: true
categories: "Node"
---

## VSCode环境设置

### 安装Debugger for Chrome插件

打开VSCode的扫描件安装面板搜索`debugger for chrome`，下图所示：

![search-result-of-debugger-for-chrome](/images/search-result-of-debugger-for-chrome.png)

## 配置debugger for chrome插件
1, 点击`debugging`图标，打开**debug**设置面板

2, 点击`play`旁边的下拉框，选择"Add Configuration …"

3, 选择"Select Environment"下拉框中的"Chrome"

{% localhost config-debugger-for-chrome.mp4 480 200 %}

上面的一系列操作实际上是在你的项目文件夹里创建了一个新的`.vscode`文件夹，这个文件夹包含`launch.json`文件。
文件内容如下：
``` javascript
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome against localhost",
      "url": "http://localhost:8080",
      "webRoot": "${workspaceRoot}"
    }
  ]
}
```
修改文件中的`url`属性为`http://localhost:3000`

## 运行Debugger

### 启动Debugger
* 按下F5
* 按下绿色的debug面板play按钮
* 点击`Debug->Start Debugger`

### 设置断点
1, 打开文件`src/App.js`
2, 在点击11行前面的空白区，出现红色的点说明已经断点设置成功
如下图

{% localhost set-debug.mp4 480 340 %}

刷新一下[ http://localhost:3000/]( http://localhost:3000/)，这时会看到页面会终止，是因为我们在VSCode设置的断点起作用了。
