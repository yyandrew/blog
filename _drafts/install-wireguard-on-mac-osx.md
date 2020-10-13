---
layout: post
title: MacOS安装WireGuard客户端
category: Tools
---

# MacOS安装WireGuard客户端

旧的MacOS系统在支持WireGuard，如果你的系统是macOS High Sierra (10.13)或者更老，请先升级。

## 安装WireGuard

使用brew工具安装：

```
brew install wireguard-tools
```

## 配置WireGuard

将配置文件拷贝到指定目录。

```
# 将配置文件拷贝到WireGuard
mkdir /usr/local/etc/wireguard/
cp <username>.conf /usr/local/etc/wireguard/wg0.conf

# 启动WireGuard VPN
sudo wg-quick up wg0

# 验证WireGuard是否正常工作
sudo wg

# 查看ip地址是否已经更改
brew install jq
curl -s https://ipvigilante.com/$(curl -s https://ipinfo.io/ip) | jq
```