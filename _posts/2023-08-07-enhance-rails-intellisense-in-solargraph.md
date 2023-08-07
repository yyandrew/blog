---
layout: post
title: 提升solargraph里的rails方法的智能提示
date: 2023-08-07 14:18 +0800
---
### 环境
1. 编辑器 Neovim
2. Neovim 的 ruby lsp 客户端
2. solargraph v0.44.0

### 配置安装 Neovim 的 ruby lsp客户端
按照[官方文档](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#solargraph)进行安装
### 安装 solargraph gem
```
gem install solargraph -v 0.44.0
```
### 生成 ruby 的官方 api 的文档
进入到 Rails 的项目，运行下面的命令
```
solargraph download-core
```
此时编辑器会提示 ruby 的内置方法的文档
### 生成 Rails 文档
运行下面命令为当前 Rails 项目的各个插件生成文档
```
bundle exec yard gems
```
### 添加配置文件使用 Neovim 提示文档
将 [https://gist.github.com/castwide/28b349566a223dfb439a337aea29713e](https://gist.github.com/castwide/28b349566a223dfb439a337aea29713e) 保存到 Rails 项目的 rails.rb 文件中

### 如果还没有提示
可以尝试下面命令
```
solargraph clear # 清除之前的文档
bundle exec yard gems --rebuild # 重新生成文档
```

参考链接
1. https://github.com/castwide/solargraph/issues/87
