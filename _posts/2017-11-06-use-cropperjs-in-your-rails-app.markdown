---
layout: post
title: "在Rails应用中使用cropperjs修剪图片"
date: 2017-11-06 18:32:32 +0800
comments: true
categories: "Rails"
---

## 准备工作
1, 从[github](https://github.com/fengyuanchen/cropperjs)上下载zip压缩包

2, 解压到任意目录

3, 将dist/cropper.min.js和dest/cropper.min.css两个文件分别复制到`项目路径/vendor/assets/javascripts/`目录和`项目路径/vendor/assets/stylesheets/`目录下

## 修改Rails配置文件
1，将`//= require cropper.min`添加到文件`application.js`最后一列

2，将`*= require cropper.min`添加到`/application.css.scss`里的最后一行

到此为止你成功引入了`cropperjs`所需的所有文件，你可以在项目中使用cropperjs的功能了。
