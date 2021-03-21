---
layout: post
title: 解决Ubuntu安装package时报错The following packages have unmet dependencies error
date: 2021-03-21 22:46 +0800
---

错误信息如下:

```shell
$ sudo apt install okular               
[sudo] password for andrew: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 qml-module-qtquick-dialogs : Depends: qml-module-qtquick-privatewidgets but it is not going to be installed
                              Depends: libqt5qml5 (< 5.12.9~) but 5.14.2+dfsg-3ubuntu1 is to be installed
                              Depends: qtbase-abi-5-12-8
                              Depends: qtdeclarative-abi-5-12-8
E: Unable to correct problems, you have held broken packages.
```

解决方法：重新安装正确的版本的package.

操作步骤：

1. 使用`apt-cache madison package`查看可用的依赖可用的版本;
2. 使用`sudo apt reinstall package=version`重新安装依赖.

示例：

```
apt-cache madison libqt5qml5 # libqt5qml5 | 5.12.8-0ubuntu1 | http://ftp.sjtu.edu.cn/ubuntu focal/universe amd64 Packages
sudo apt reinstall libqt5qml5=5.12.8-0ubuntu1
```

