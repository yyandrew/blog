---
layout: post
title: "常用*nix命令"
date: 2015-10-22 11:50
comments: true
categories: "System"
---
* 显示硬盘使用状态
      df -h --total
* 显示文件夹或者文件大小
      # 显示当前文件夹大小
      du -s -h .
      # 显示file1大小
      du -s -h file1
      # 显示当前文件夹下文件及文件夹大小
      du -s -h ./*
* 显示当前文件夹及其子文件夹大于20M的文件
      find / -type f -size +20000k -exec ls -lh {} \; | awk '{ print $9 ": " $5 }'
