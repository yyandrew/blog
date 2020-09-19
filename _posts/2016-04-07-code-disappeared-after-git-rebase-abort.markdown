---
layout: post
title: "git rebase --abort造成的代码丢失"
date: 2016-04-07 14:51
comments: true
categories: "Git"
---
事情的经过是这样的，本人像平常一下准备将做完的代码push到远程
``` bash
$ git pull origin staging
```
结果等待我的是
```
git push origin stagingFrom git.example.com:app
 * branch            staging    -> FETCH_HEAD

It seems that there is already a rebase-apply directory, and
I wonder if you are in the middle of another rebase.  If that is the
case, please try
        git rebase (--continue | --abort | --skip)
If that is not the case, please
        rm -fr "/Users/andrew/Development/sonar/.git/rebase-apply"
```
什么？怎么会报错？不管先终止rebase再说
``` bash
$ git rebase --abort
```
再重新做pull
``` bash
$ git pull origin staging
```
结果如下
``` bash
From git.example.com:app
 * branch            staging    -> FETCH_HEAD
Current branch staging is up to date.
```
貌似一切正常.然后做push代码到远程
``` bash
$ git push origin staging
```
结果如下
```
Everything up-to-date
```
什么？没有代码被push。预感不好，要跪，赶紧`git log`，我的commit不见了，感觉天要塌下来了:(
赶紧救助google。终于找到一些[解决方法](http://stackoverflow.com/a/2693668)
``` bash
$ git reflog | grep 50554 # 50554是commit信息的关键字
def8c08 HEAD@{5}: commit: FEEDBACK #50554
$ git reset --hard HEAD@{5}
HEAD is now at def8c08 FEEDBACK #50554
```
[git reflog](https://git-scm.com/docs/git-reflog)建功了:)我的代码也回来了。
不得不说git真的非常的强大。只要不运行`git-gc`你的代码就会找回来。
