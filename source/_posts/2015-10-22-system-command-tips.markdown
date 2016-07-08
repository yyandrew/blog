---
layout: post
title: "常用*nix命令"
date: 2015-10-22 11:50
comments: true
categories: "System"
---
* 显示硬盘使用状态
```bash
df -h --total
```
* 显示文件夹或者文件大小
```bash
# 显示当前文件夹大小
du -s -h .
# 显示file1大小
du -s -h file1
# 显示当前文件夹下文件及文件夹大小
du -s -h ./*
# 计算~/test_dir目录下文件的数量
find ~/test_dir -type f | wc -l
```
* 删除所有空文件夹
```bash
find . -type d -empty -delete
```
* 显示当前文件夹及其子文件夹大于20M并且小于30M的文件
```bash
find / -type f -size +20M -size -30M -exec ls -lh {} \; | awk '{ print $9 ": " $5 }'
```
* 显示所有被监听的端口及其程序
```bash
sudo netstat -tulpn # t指tcp协议, u指udp协议,l指指显示被监听,p指显示程序名称,n指以ip地址+端口显示地址而不是以字符显示
```
* 生成公钥
```bash
ssh-keygen -t rsa # 一路回车到底
cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"
```
* 随机生成32位密码
```bash
openssl rand -base64 32
```
* 生成100M的大文件
```bash
fallocate -l 100M test.img
```
* rsync备份系统

``` bash
rsync -azvP /etc/* username@remote_host:/backup/ #注意敏感文件的权限
```

* iptables操作
```bash
sudo iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT # 添加一个开放80端口的规则
sudo iptables -I INPUT -s x.x.x.x -j DROP # 阻止ip为x.x.x.x的所有访问
sudo iptables -nL --line-numbers # 显示所有规则并带行号
sudo iptables -D INPUT 5 # 删除INPUT链第5行处的规则
sudo iptables -I INPUT 5 -i lo -p tcp -m tcp --dport 80 -j ACCEPT # 将开放80端口的规则插入到第5行
sudo iptables -A INPUT -m mac --mac-source 00:0F:EA:91:04:08 -j DROP # 阻止MAC地址为00:0F:EA:91:04:08的所有访问
sudo /etc/init.d/iptables save # 使修改生效
sudo iptables-save > /root/rule.file # 将当前防火墙规则保存到/root/rule.file
sudo iptables-restore < /root/rule.file # 使用/root/rule.file的规则
```
* mysql操作
```bash
$ mysql # 进入mysql控制台
mysql> select * from mysql.user; # 显示所有用户
mysql> create database your_db_name;
mysql> SET PASSWORD FOR 'user'@'source_ip' = PASSWORD('password'); # 允许user从source_ip远程登录mysql
mysql> grant all privileges on db1.* to 'user'@'source_ip'; # 允许user从source_ip管理db1数据库
mysql> revoke all privileges on db1.* from 'user'@'source_ip'; # 禁止user从source_ip管理db1数据库
mysql> drop user 'user'@'source_ip'; # 删除user从source_ip管理db1数据库
mysql> grant usage on *.* to your_user@localhost identified by 'your_user_password'; # 创建新用户
mysql> grant all privileges on your_db_name.* to your_user@localhost ; 设置database的所有者

# 备份及恢复备份
$ mysql database_name < db_back.sql # 导入一个sql备份文件
$ mysql database_name > db_back.sql # 备份database_name数据库
```
* 上传id_rsa.pub之后ssh免密码登录失效的另类解决方法
```bash
$ chmod 600 ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/id_rsa
$ chmod 700 ~/.ssh/id_rsa.pub
$ chmod 700 ~/.ssh
```
* 测试linux服务器的性能
```bash
$ wget -qO- bench.sh | bash
```

* pure-ftps使用(centos)
```bash
# 基本操作
service pure-ftpd start # 启动pure-ftpd服务
service pure-ftpd stop # 停止pure-ftpd服务
# 修改已存在用户的密码
pure-pw passwd username
pure-pw mkdb
```
