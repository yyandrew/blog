---
layout: post
title: 怎么调试 Node.js 项目
date: 2021-06-30 11:51 +0800
---

1. 启动node项目.`node server.js`；
2. 通过命令 `ps aux | grep server.js` 找到 pid ；
3. 使用命令 `node inspect -p pid`  启动调试器；
4. 打开 chrome 浏览器的 DevTools 工具 ，点击面板上的 Node.js 的图标打开 DevTools - Node.js 窗口；
5. 选择 Sources 面板，在左侧 Node 选项卡找到需要调试的代码行创建一个一个断点开始调试。

![DevTools-for-Node.js.png](/images/DevTools-for-Node.js.png)
