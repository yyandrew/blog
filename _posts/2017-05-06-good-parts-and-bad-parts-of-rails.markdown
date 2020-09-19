---
layout: post
title: "Rails框架的好与坏"
date: 2017-05-06 10:14:28 +0800
comments: true
categories: Rails
---

# 首先必须得承认Rails是一个非常优秀的框架

## 我们先从它的优点开始说起吧
* 官方入门教程通俗易懂，容易入手
* 大量的gem扩展插件可以使用
* 自带增强过的HTTP头
* 自带CSRF安全机制
* ActiveRecord自带SQL防注入机制
* 不同的环境用不同的配置（development，test，production）
* console调试非常方便
* 集成email功能
* 集成job功能
* 集成database migration功能

## 接下是它的缺点
* 不能做局部的升级，只能全部升级。比如我只想使用ActionCable，你就必须要升级整个Rails，还得处理一些旧的Gem包的不兼容新的Rails的问题。
* 对Ruby cord使用Monkey patches以及元编程的滥用
* 偶尔会出现的死锁，特别是在使用ActionController::Live
* ActiveSupport::Dependencies加载的问题,你得引入全部的`action_view`，当你只想使用一下`action_view/helpers/number_helper`
* 启动太慢,spring并不能很好的解决问题
* 关于app启动过程及request生命周期的说明文档不太详细

[原文链接](https://rosenfeld.herokuapp.com/en/articles/ruby-rails/2017-05-03-ruby-on-rails-the-bad-and-good-parts)