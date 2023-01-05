---
layout: post
title: 使用Github Action自动部署github page
date: 2023-01-05 13:35 +0800
---

下面介绍一下如何使用Github Action实现自动部署博客的方法。这种方法也是本博客正在使用的自动部署的方法。
1. 在博客根目录下新建文件及文件夹`.github/workflows/deploy-workflow.yml`，内容如下:

``` yaml
name: Build and Deploy
# 当新提交被推送到master分支时自动触发此action
on:
  push:
    branches:
      - master
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          persist-credentials: false
      - name: Deploy Jekyll Site
        # 开始生成静态文件及并将最新的修改推送到gh-pages分支
        uses: yyandrew/github-pages-deploy-action-for-jekyll@v2
        env:
          # 下面三个环境变量会在更新gh-pages分支时使用到
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITHUB_REPOSITORY: ${{ secrets.GITHUB_REPOSITORY }}
          GITHUB_ACTOR: ${{ secrets.GITHUB_ACTOR }}
```

{:start="2"}
2. 以后每次更新master分支的代码就会自动更新gh-pages分支了
3. 如何自定义github pages的域名?
    {:start="1"}
    1. 在根目录新建CNAME文件，将你的域名添加进去
    2. 将你的域名填到[settings/pages](https://github.com/yyandrew/blog/settings/pages)的**Custom domain**
    3. 在域名管理页面新增一个CNAME的记录。Name为blog，content为yourname.github.io
