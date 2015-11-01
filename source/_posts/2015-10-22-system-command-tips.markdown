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
* 删除所有空文件夹
      find . -type d -empty -delete
* 显示当前文件夹及其子文件夹大于20M并且小于30M的文件
      find / -type f -size +20M -size -30M -exec ls -lh {} \; | awk '{ print $9 ": " $5 }'
* 显示所有被监听的端口及其程序
      sudo netstat -tulpn # t指tcp协议, u指udp协议,l指指显示被监听,p指显示程序名称,n指以ip地址+端口显示地址而不是以字符显示
* 生成公钥
      ssh-keygen -t rsa # 一路回车到底
      cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"
