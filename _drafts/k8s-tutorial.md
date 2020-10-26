### 通过vagrant安装虚拟机
下面所有的操作都会在这个虚拟机里执行。
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "hashicorp/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = "2048" # kubernetes需要至少2GB内存
    vb.cpus = 2 # kubernetes需要至少2核CPU
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
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
    # 配置kubeadm
    kubeadm init
    sudo cp /etc/kubernetes/admin.conf $HOME/
    sudo chown $(id -u):$(id -g) $HOME/admin.conf
    export KUBECONFIG=$HOME/admin.conf
    # 安装minikube
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
    dpkg -i minikube_latest_amd64.deb
  SHELL
end
```

### 启动，构建环境并进入虚拟机

```shell
vagrant up
vagrant provisor
vagrant ssh
```

### 配置kuberadmin

用来创建kubernetes集群的工具。

```shell
kubeadm init
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
```

### 启动minikube

使用minikube，你可以在本地轻松的搭建好kubernetes环境。

```sh 
sudo usermod -aG docker $USER && newgrp docker
minikube start --driver=docker
```

如果输出如下内容表示minikube启动成功，如果启动不成功能的话一般是被卡住。

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

### 创建一个nginx示例：

新建一个文件`my-nginx-rc.yaml`，内容如下：

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

1. 创建名为`my-nginx`（由.metadata.name指定）的Deployment
2. 创建2个（由`spec.replicas`字段标明）Pod副本
3. `selector`字段定义Deployment如何快速查找要管理的Pods。
4. `template`p字段包含以下字段
   1. Pod被使用`labels`字段打上`run: my-nginx`
   2. Pod模板规约（即`template.spec`字段）指示Pods运行一个`nginx`容器。该容器运行最新版本的`nginx`Docker Hub镜像。
   3. 创建一个容器并使用`name`字段将其命名为`my-nginx`。

使用kubectl工具创建pods

```shell
kubectl create -f my-nginx-rc.yaml
```

使用kubectl工具将pods暴露出来

```sh 
kubectl expose deployment/my-nginx --type=NodePort
```

查看service的url

```
minikube serivce my-nginx --url
# http://192.168.49.2:32328
```

访问nginx

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
```

参考链接

1. https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/