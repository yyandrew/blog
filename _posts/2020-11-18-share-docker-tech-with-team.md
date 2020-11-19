---
layout: post
title: 团队分享Docker技术演讲草稿
date: 2020-11-18 17:24 +0800
category: Docker
---

## 如何使用Docker搭建Rails开发环境

### 第0页

![docker-rails-page-0.png](/images/docker-rails-page-0.png)

这次Workshop的主要内容有
1. 介绍什么是Docker
2. 怎么使用Docker
3. 怎么使用Dockerfile定制镜像
4. 介绍Docker Compose
5. 操作演示。从零开始搭建Rails开发环境

## 第1页

![docker-rails-page-1.png](/images/docker-rails-page-1.png)
首先大家要明白Docker只是一个工具。Docker通过将应用程序运行时所需析软件及资源打包到容器里来简化我们创建，部署，运行应用的工作。

### 第2页

![architecture-of-docker.png](/images/architecture-of-docker.png)

Docker的结构
Docker使用典型的C/S结构。客户端发送docker指令，Docker daemon监听指令请求，对请求作反应。比如管理镜像，容器，网络，匿名卷等。

这张图包含了一些Docker的基本概念：
1. 镜像 - Docker镜像是一个特殊的文件系统除了提供容器运行时所需的程序、资源、配置文件 外，还包含了一些运行时准备的一些配置参数(如环境变量、用户等)。镜像不包含任何动态数据、其内容构建之后不会被改变。
2. 容器 - 容器是镜像运行时的实体。容器可以被创建、启动、停止、删除暂停等。镜像和容器的关系就像程序和进程的关系。
3. 仓库 - 镜像构建完成之后，可以很容易的在当前宿主机上运行。但是需要在其它的服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务。Docker registry就是这样的服务。一个Docker Registry中可以有多个仓库，每个仓库可以有多个标签，每个标签对应一个镜像。

### 第3页

![connect-each-component-of-docker.png](/images/connect-each-component-of-docker.png)

接下来我们通过一个例子讲解一下各个组件如何协同工作。
以`docker run`为例子，当你运行这条命令的时候，docker究竟做了哪些事？
首先，Docker会将我们需要的镜像准备好。在这个例子里镜像是nginx
接下来创建一个容器。
然后是为这个容器分配文件系统以便用户可以访问容器里的文件。
再接下来是为容器分配网络接口，这样容器就可以和外界通信了。
最后是启动容器并且运行`bash`命令。这条命令之后我们就可以使用控制台终端和容器交互了。

### 第4页

![how-to-use-docker.png ](/images/how-to-use-docker.png)

这一页展示了如何使用Docker搭建一个Nginx服务器。同时也展示了Docker的完整的生命周期。
第一步使用docker pull下载nginx镜像
第二步使用docker run运行镜像。-p表示宿主机和容器端口的映射关系。这里80:80表示将容器的80端口映射到宿主机的80端口。-v表示创建一个共享目录，以便宿主机和容器进行文件共享。-w表示设置一个工作目录。--name表示为容器指定一个名字，便于记忆。最后的一个值nginx表示使用哪个镜像。这里是nginx。
第三步使用curl命令测试。
第四步打开控制台终端等待用户输入命令和容器进行交互。
第五步删除容器
第六步删除镜像

### 第5页
![whats-dockerfile.png](/images/whats-dockerfile.png)
下面我们介绍一下Dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，这些指令描述了镜像的创建过程。

使用 Dockerfile 使镜像构建文档化，不仅仅开发团队可以理解应用运行环境，也方便运维团队理解应用运行所需条件，帮助他们更好的在生产环境中部署该镜像。

### 第6页
![how-to-build-rails-image.png](/images/how-to-build-rails-image.png)

以定制Rails开发环境镜像为例子。
1. FROM 指定基础镜像
2. RUN 指令是用来执行命令行命令
3. WORKDIR 指定工作目录（或者称为当前目录）
4. COPY 复制文件。格式为`COPY <宿主机源路径>... <镜像内目标路径>`
5. EXPOSE 指令是声明运行时容器提供服务端口
6. CMD 指令就是用于指定默认的容器主进程的启动命令的。由于Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。

### 第7页
![introduce-docker-compose.png](/images/introduce-docker-compose.png)
Docker Compose是Docker官方编排工具之一，可以通过它定义多容器docker应用

### 第8页

![introduce-docker-compose-yml.png](/images/introduce-docker-compose-yml.png)

模板文件是使用 Compose 的核心，涉及到的指令关键字也比较多。但大家不用担心，这里面大部分指令跟docker run 相关参数的含义都是类似的。

每个服务都必须通过 image 指令指定镜像或 build 指令（需要 Dockerfile）等来自动构建生成镜像。

1. build 指定 Dockerfile 所在文件夹的路径
2. ports 暴露端口信息。使用宿主端口：容器端口 (HOST:CONTAINER) 格式
3. volumes 数据卷所挂载路径设置。可以设置为宿主机路径(HOST:CONTAINER)
4. depends_on 解决容器的依赖、启动先后的问题。以下例子中会先启动postgres再启动web。

### 第9页
![practice-of-dockerize-rails-app.png](/images/practice-of-dockerize-rails-app.png)

使用Docker创建一个Rails API，并添加一个users的endpoint。
要求：

1. 数据库用postgres:9.6-alpine（体积小，下载速度快）
2. 使用docker-compose编排服务

只有20分钟练习。

详细的操作说明可以查看[github repo](https://github.com/yyandrew/dockerize-rails-app/tree/14e84e49c88bd9d162b75bbfe16e2903797b5f9b)

