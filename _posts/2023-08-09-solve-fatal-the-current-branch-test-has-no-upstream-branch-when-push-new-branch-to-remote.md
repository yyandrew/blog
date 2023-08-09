---
layout: post
title: '解决git里推送新分支时出现的烦人 "fatal: The current branch test has no upstream branch"提示'
date: 2023-08-09 09:49 +0800
---
### 问题描述
你如果在推送一个新的分支时出现下面提示的话就可以使用本文里的方法解决它。

```
fatal: The current branch test has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin test

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.
```
### 解决方法
将下面内容添加到 git 全局配置文件`~/.gitconfig`，可以运行`git config --global --edit`命令在系统默认编辑器里打开它。
```
[push]
	autoSetupRemote = true

```
再重新推送分支，是不是没有提示并且成功推送分支了。
