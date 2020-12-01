---
layout: post
title: 容器化solargraph
date: 2020-12-01 14:15 +0800
category: Tools
---

涉及到的工具:

1. [solargraph](https://github.com/castwide/solargraph) - 提供了ruby编程的工具：智能提示，代码补全，内置文档等
2. [neovim](https://neovim.io/) - 古老的文本编辑器vim的强化版
3. [neoclide/coc.nvim](https://github.com/neoclide/coc.nvim) - neovim插件，支持各种开源协议 LSP
4. [neoclide/coc-solargraph](https://github.com/neoclide/coc-solargraph) - Solargraph extension for coc.nvim
5. docker
6. docker-compose

### 构建docker镜像

```
# Dockerfile
from ruby:2.7.2 # 指定基础镜像
run gem install solargraph # 安装solargraph gem

expose 7658 # 暴露容器的7658端口

cmd ["solargraph", "socket", "--host=0.0.0.0", "--port=7658"] # 运行容器的默认命令
```

使用`docker build -t solargraph . `构建一个名为`solargraph`的镜像。

### 运行solargraph镜像

```shell 
docker run --rm -d -p 7658:7658 --name solargraph solargraph
```

* --rm 自动删除上次的容器
* -d 使用守护进程模式运行容器
* -p 将宿主机7658端口映射到容器的7658端口
* --name 指定容器的名字为solargraph，便于记忆
* 最后的`solargraph`为指定的镜像名字

### 设置Coc

1. 安装`coc.nvim`及`coc-solargraph`的方法可能参考对应项目的README文件。

2. 在vim里使用`:CocConfig`打开`coc-settings.json`，将下面内容添加到`coc-settings.json`

```
{
  "solargraph.transport": "external",
  "solargraph.externalServer": {
    "host": "localhost",
    "port": 7658
  }
}
```

### 最终效果

![solargraph-in-vim.png](/images/solargraph-in-vim.png)

