---
layout: post
title: "常用*nix命令"
date: 2015-10-22 11:50
comments: true
categories: "System"
---
* 显示硬盘使用状态

``` bash
df -h --total
```

* 测试硬盘IO速度

``` bash
dd if=/dev/zero of=/tmp/output conv=fdatasync bs=384k count=1k; rm -f /tmp/output
```

* 显示文件夹或者文件大小
``` bash
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
``` bash
find . -type d -empty -delete
```
* 显示当前文件夹及其子文件夹大于20M并且小于30M的文件
``` bash
find / -type f -size +20M -size -30M -exec ls -lh {} \; | awk '{ print $9 ": " $5 }'
```

* 查找所有excel文件
``` bash
find . -type f -name *.xlsx
```

* 显示所有被监听的端口及其程序
``` bash
sudo netstat -tulpn # t指tcp协议, u指udp协议,l指指显示被监听,p指显示程序名称,n指以ip地址+端口显示地址而不是以字符显示
```
* 生成公钥
``` bash
ssh-keygen -t rsa # 一路回车到底
cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"
```
* 随机生成32位密码
``` bash
openssl rand -base64 32
```
* 生成100M的大文件
``` bash
fallocate -l 100M test.img
```
* rsync备份系统

``` bash
rsync -azvP /etc/* username@remote_host:/backup/ #注意敏感文件的权限
```

* iptables操作
``` bash
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

``` bash
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
``` bash
$ chmod 600 ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/id_rsa
$ chmod 700 ~/.ssh/id_rsa.pub
$ chmod 700 ~/.ssh
```
* 测试linux服务器的性能
``` bash
$ wget -qO- bench.sh | bash
```

* pure-ftps使用(centos)

``` bash
# 基本操作
service pure-ftpd start # 启动pure-ftpd服务
service pure-ftpd stop # 停止pure-ftpd服务
# 修改已存在用户的密码
pure-pw passwd username
pure-pw mkdb
```

* 获取当前ip地址
``` bash
ifconfig | sed -En 's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p'
```

* 备份及还原gpg keys
``` sh
# 备份
cp ~/.gnupg/pubring.gpg /path/to/backups/
cp ~/.gnupg/secring.gpg /path/to/backups/
cp ~/.gnupg/trustdb.gpg /path/to/backups/
# or, instead of backing up trustdb...
# gpg --export-ownertrust > chrisroos-ownertrust-gpg.txt

# 还原
cp /path/to/backups/*.gpg ~/.gnupg/
# or, if you exported the ownertrust
# gpg --import-ownertrust chrisroos-ownertrust-gpg.txt
```

* 显示使用频率最高的前10的Linux命令

``` sh
history | awk '{CMD[$2]++;count++;} END { for (a in CMD )print CMD[ a  ]" " CMD[ a  ]/count*100 "% " a  }' | grep -v "./" | column -c3 -s " " -t |sort -nr | nl | head -n10
```

* CURL命令

``` sh
# 在`curl`命令中添加用户参数信
curl -X POST -u "username:password" -F file=@/file_path/Exemple-export.xml http://localhost:3000/upload

# 发送一个put请求并且带上参数
curl -i -X PUT http://localhost:3000/games/5920624ebfda25d15fc6486e --data '{"title": "abc123", "cover": "abc123"}' -H "Content-Type: application/json
```

* 修改默认的编辑器

``` sh
sudo update-alternatives --config editor
```

* 检查所有有ssh连接

``` sh
netstat -algrep ssh
```

* 升级mac os的node

``` sh
sudo npm cache clean -f
sudo npm install -g n
sudo n stable
node -v # 检查版本号
```

* apt命令使用

``` sh
apt-cache pkgnames # 查看操作系统所有可以使用的工具
apt-cache search nginx # 查看nginx的名字和简要ras描述
apt-cache showpkg nginx # 查看nginx的依赖
apt-cache show nginx # 查看nginx详细信息
apt-cache stats # 查看cache的状态
apt-get install nginx # 安装或者更新nginx, 加上‘–no-upgrade’可以不更新
apt-get install nginx --only-upgrade # 只更新nginx
apt-get install nginx=1.2.1-2.2+wheezy4 # 只安装1.2.1-2.2+wheezy4的nginx
apt-get purge nginx # 卸载nginx，包含配置文件
apt-get clean # 删除下载的.deb文件，释放硬盘空间
apt-get --download-only source nginx # 只下载源码
apt-get source nginx # 下载源码并解压
apt-get --compile source nginx # 下载源码，解压，编译
apt-get download nginx # 仅仅下载
apt-get changelog nginx # 查看更新日志
apt-get check # 查看是否有损坏依赖包
apt-get build-dep nginx # 搜索安装nginx的依赖包
apt-get autoclean # 清空cache
```

# 根据关键字终止进程
```sh
kill -9 $(ps aux | grep 'git' | awk '{print $2}') # 查出所有git进程，并终止它们
```

# 断点下载大文件的方法
```sh
rsync -P -e ssh user@host:remote_file local_file
```

# grep使用方法
```sh
sudo grep -Rw / -e 'check_root' # R - all files under each directory, recursively, following symbolic links, w - select only those lines containing matches
```