---
layout: post
title: "Kubernetes入门"
date: 2020-10-27 14:24:05 +0800
categories: "DevOps"
---

## 什么是Kubernetes

Kubernets（简称K8s）是一个全新的基于容器技术的分布式架构领先方案。k8s具有完备的集群管理能力，包括多层次安全防护和准入机制、多租户应用支撑能力、透明的服务注册和服务发现机制、内建的智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调试机制、以及多粒度资源配额管理能力。同时k8s提供了完善的管理工具，这些工具涵盖了包括开发、部署测试、运维监控在内的各个环节。因此，k8s是一个全新的基于容器技术的分布式架构解决方案，并且是一个一站式的完备的分布式系统开发和支撑平台。

## 为什么要用Kubernetes

使用k8s的理由有很多，最重要的的理由是，IT行业从来都是由新技术驱动的。

使用k8s收获的好处：

1. 可以“轻装上阵”地开发复杂系统。采用k8s解决方案之后，只需要一名架构师负责系统中服务组件的架构设计，几名开发工程师负责业务代码的开发，一名系统兼运维工程师负责k8s的部署和运维。
2. 可以全面拥抱微服务架构。
3. 可以随时将系统整体“搬迁”到公有云上。
4. k8s内在的服务弹性扩容机制可以让我们轻松应对突发流量。在服务高峰期，我们可以选择在公有云中快速扩容某些service的实例副本以提升系统的吞吐量。
5. k8s系统架构超强的横向扩容能力可以让我们的竞争力大大提升。

## 基本概念和术语

#### Master

k8s里的Master指的是集群控制节点，在每个k8s集群里都需要有一个Master来负责整个集群的管理和控制。

在Master上运行着以下关键进程：

1. Kubernetes API Server：提供了HTTP Rest接口的关键服务进程，是k8s里所有资源的增、删、改、查等操作的唯一入口。
2. Kubernetes Controller Manager：k8s里所有资源对象的自动化控制中心，可以将其理解成资源对象的“大总管”
3. Kubernetes Scheduler：负责资源调度（Pod调度）的进程，相当于公交公司的“调度室”

#### Node

 除了Master，k8s集群中的其他机器被称为Node。与master一样，Node可以是一台物理主机，也可以是一台虚拟机。每个Node被master分配一些工作负载（Docker容器），当某个Node宕机时，其上的工作负载会被master自动转移到其他节点上。

在每个Node上都运行着以下关键进程：

1. kubelet：负责Pod对应的容器的创建、启动停等任务，同时与Master密切协作，实现集群管理的基本功能。
2. kube-proxy：实现k8s Service的通信与负载均衡机制的重要组件
3. Docker Engine：Docker引擎，负责本机的容器创建和管理工作。

#### Pod

Pod是k8s最重要的概念。下图是Pod的组成示意图。

![pods-structure.png](/images/pods-structure.png)

我们可以看到每个Pod都有一个被称为“根容器”的Pause容器，除了Pause容器，每个Pod还包含一个或多个紧密相关的用户业务容器。

Pod运行在一个被称为节点(Node)的环境中，这个节点既可以是物理机，也可以是私有云或者公有云中的一个虚拟机，通常在一个节点上可以运行几百个pod；其次，在每个pod中都运行着一个特殊的容器（被称作Pause容器）和其他的业务容器，这些业务容器共享Pause容器的网络栈和Volumn，因此它们之间的通信和数据交换更为高效，在设计时我们可以充分利用这一特性将一组密切相关的服务进程放入同一个pod中。

#### Label

一个Label是一个key=value的键值对，其中key与value是用户自己指定。label可以被附加各种资源对象上，例如Node、Pod、Service、RC等。

#### Replication Controller

RC是k8s系统的核心概念之一。简单的来说，它其实定义了一个期望的场景，即声明某种Pod的副本数量在任意时刻都符合某个预期值，所以RC的定义包括如下几个部分：

1. Pod期待的数量
2. 用于筛选目标Pod的Label Selector
3. 当Pod的副本数量小于预期数量时，用于创建新Pod的Pod模板。

#### Deployment

解决Pod的编排问题。是Replication Controller的一次升级，两者相似度超过90%

#### Horizontal Pod Autoscaler

k8s里的Pod智能扩容的特性。通过分析指定的RC控制的所有目标Pod的负载变化情况，来确定是否需要有针对性地调整目标 Pod的副本数量，这是HPA的实现原理。当前，HPA有以下两种方式作为Pod负载的度量指标：

1. CPUUtilizationPercentage - 目标Pod所有副本自身的CPU利用率的平均值。
2. 程序自定义的试题指标，比如服务在每秒内的相应请求数（TPD或QPS）

#### Service

Service是分布式集群架构的核心，service的服务进程目前都基于Socket通信方式对外提供服务。一个service对象拥有如下关键特征：

1. 拥有唯一的名称(比如my-nginx)
2. 拥有一个虚拟IP（Cluster IP、Service IP或VIP）和端口号（比如192.168.49.2:32328）
3. 能够提供某种远程服务能力（比如上面的deployment/my-ngin提供了Web Server服务）
4. 被映射到提供这种服务能力的一组容器应用上（比如上面的deployment/my-ngin）

下图显示了Pod，RC与Service的逻辑关系

![pod-rc-and-service.jpg](/images/pod-rc-and-service.jpg)

从上图可以看出k8s的Service定义了一个服务的访问入口地址，前端的应用（Pod）通过这个入口地址访问其背后的一组由Pod副本组成的集群实例，Service与其后端Pod副本集群之间则是通过Label Selector来实现无缝对接的。

#### Volume

Volume是Pod中能够被多个容器访问的共享目录。

#### Namespace

Namespace在很多情况下用于实现多租户资源隔离。

## 从一个简单的例子开始

下面是一个利用k8s部署一个nginx的例子。

需要用到的工具：

1. [vagrant](https://www.vagrantup.com/downloads)
2. [virtualbox](https://www.virtualbox.org/wiki/Downloads)

#### 通过vagrant安装虚拟机

下面所有的操作都会在这个虚拟机里执行。
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096" # kubernetes需要至少2GB内存
    vb.cpus = 2 # kubernetes需要至少2核CPU
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update && apt-get install -y apt-transport-https curl
    # 安装docker
    apt-get remove docker docker-engine docker.io containerd runc
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    apt-key fingerprint 0EBFCD88
    add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
    apt-get update
    apt-get install -y docker-ce docker-ce-cli containerd.io
    # 安装 kubeadm、kubelet 和 kubectl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<-EOF | tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
    apt-get update
    apt-get install -y kubelet kubeadm kubectl
    apt-mark hold kubelet kubeadm kubectl
    # 安装minikube
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
    dpkg -i minikube_latest_amd64.deb
  SHELL
end
```

#### 启动，构建环境并进入虚拟机

```shell
vagrant up
# vagrant provision
vagrant ssh
```

#### 配置kuberadmin

用来创建kubernetes集群的工具。

```shell
sudo swapoff -a # 关闭Swap分区，kubeadm不支持运行在为Swap分区的机器上
sudo kubeadm init
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

#### 启动minikube

使用minikube，你可以在本地轻松的搭建好kubernetes环境。

```sh 
sudo usermod -aG docker $USER && newgrp docker
minikube start --driver=docker
minikube status # 查看是minikube否已经启动
```

如果输出如下内容表示minikube启动成功。

PS：一定要注意看运行minikube的硬件要求（CPU，内存不满足要求的话会出错）

```
vagrant@vagrant:~$ minikube start --driver=docker
* minikube v1.14.0 on Ubuntu 18.04 (vbox/amd64)
* Using the docker driver based on existing profile

X Requested memory allocation (985MB) is less than the recommended minimum 1907MB. Deployments may fail.

* Starting control plane node minikube in cluster minikube
* Restarting existing docker container for "minikube" ...
* Preparing Kubernetes v1.19.2 on Docker 19.03.8 ...
* Verifying Kubernetes components...
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" by default
```

#### 创建Pods

一、新建一个文件`my-nginx-rc.yaml`，内容如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```

上面是Deployment示例。其中创建了一个ReplicaSet，负责启动2个Pods。除此之外，该例子还：

1. 创建名为`my-nginx`（由`metadata.name`指定）的Deployment
2. 创建2个（由`spec.replicas`字段标明）Pod副本
3. `selector`字段定义Deployment如何快速查找要管理的Pods。
4. `template`p字段包含以下字段
   1. Pod被使用`labels`字段打上`run: my-nginx`
   2. Pod模板规约（即`template.spec`字段）指示Pods运行一个`nginx`容器。该容器运行最新版本的`nginx`Docker Hub镜像。
   3. 创建一个容器并使用`name`字段将其命名为`my-nginx`。

二、使用kubectl工具创建pods

```shell
kubectl create -f my-nginx-rc.yaml
```

三、使用kubectl工具将pods暴露出来

```sh 
kubectl expose deployment/my-nginx --type=NodePort
```

四、查看Service的url

```
minikube service my-nginx --url
# http://192.168.49.2:32328
```

五、访问nginx

```shell 
curl http://192.168.49.2:32328
```

输出结果为：

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

从上面的输出可以看出nginx已经正常工作了。

### 其它命令

```
# 删除一个一组pods
kubectl delete -f my-nginx-rc.yaml
# 查看pods,deployments,services
kubectl get pods,deployments,services
# 查看service的详细信息
kubectl describe service/my-nginx
# 进入一个容器
kubectl exec my-nginx-5b56ccd65f-p6mmf -i -t bash 
```

参考链接

1. https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/

