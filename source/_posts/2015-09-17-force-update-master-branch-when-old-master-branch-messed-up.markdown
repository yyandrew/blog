---
layout: post
title: "利用新分支替换旧master分支"
date: 2015-09-17 12:37
comments: true
categories: "Git"
---
如果你原来的master分支一不小心被你玩坏了，你可以用下面的方法强行更新已经坏掉的master分支.
```ruby
git branch -m master old_master
git branch -m new_branch master
git push -f origin master
```
