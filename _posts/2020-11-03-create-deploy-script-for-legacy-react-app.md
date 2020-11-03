---
layout: post
title: 为遗留React项目创建自动部署脚本
date: 2020-11-03 14:01 +0800
category: DevOps
---

### 背景
最近公司接到一个项目，它是一个遗留项目。这个项目原先的开发人员已经被离职 :dog:。项目的部署的文件已经找不到了:cry:。本着自力更生的精神，我们试着重新构建了自动部署脚本。

### 我们已经掌握的资源
1. 遗留项目的源代码
2. 遗留项目运行的服务器



### 开始调查

在检查源代码和询问客户，确认没有可能找到之前的自动部署脚本之后。我们开始着手恢复部署脚本。

首先登录到项目运行服务器上看看。能不能找到什么。

查看nginx的配置，找到项目的根目录，查看其文件夹结构，结构如下：

```
> $ ls -al                                                                                                                                                                                                                                   
total 24
drwxrwx---  5 ansible nginx   4096 Sep 30 11:29 .
drwxr-xr-x  6 root    root    4096 Jan  2  2020 ..
lrwxrwxrwx  1 ansible ansible   26 Sep 30 11:29 current -> ./releases/20200930092703Z
-rwxrwx---  1 ansible nginx   4096 Feb 19  2020 git_identity_key
drwxrwx--- 20 ansible nginx   4096 Sep 30 11:27 releases
drwxrwx---  8 ansible nginx   4096 Sep 30 11:27 repo
drwxrwx---  2 ansible nginx   4096 Feb  6  2020 shared
```

从输出来看是使用的git工具和`ansible`色色。好像没有什么特别的。但是这里有个`git_identity_key`文件。不知道有什么用。我们搜索一下，结果如下：

![search-result-of-git_identity_key.png](/images/search-result-of-git_identity_key.png)

这个一个github的[某个repo的issue页面](https://github.com/ansistrano/deploy/issues/158)。仔细查看这个repo [ansible/deploy](https://github.com/ansistrano/deploy)。发现这是个自动部署工具。repo的描述里有Ansible role，deploy scripting等字眼，同时README文件里的部署成功后的文件夹结构和服务器上的结构一致，我们可以确定遗留项目就是用这个工具部署的。:v::v::v:

### 创建部署脚本

一、创建`.ansistrano/build.yml`

这个文件指定服务器上的代码更新之后服务器上需要执行的任务。

```yaml
- shell: cd "{{ ansistrano_deploy_to }}/current" && npm install
- shell: cd "{{ ansistrano_deploy_to }}/current" && npm run build
```

二、创建`.ansistrano/deploy.yml`

```shell
---
- name: Deploy your-app-name
  hosts: all
  vars:
    ansistrano_deploy_from: "{{ playbook_dir }}"
    ansistrano_deploy_to: "/app/folder/name"
    ansistrano_git_repo: your-git-repo-url
    ansistrano_git_branch: "master"
    ansistrano_keep_releases: 0
    # There seems to be an issue with rsync in vagrant
    ansistrano_deploy_via: "git"
    ansistrano_after_symlink_tasks_file: 'build.yml'
    ansistrano_shared_files: ['.env']
    ansistrano_git_identity_key_remote_path: ''
  roles:
    - { role: ansistrano.deploy }
```

三、创建`.ansistrano/hosts/production`

这个文件指定项目被部署到哪台服务器。详细语法可以参考[[Inventory文件](https://ansible-tran.readthedocs.io/en/latest/docs/intro_inventory.html#id7)]文档。

```
your-production-server-hostname ansible_user=ansible ansible_port=22 ansible_host=your-production-server-hostname ansible_ssh_extra_args="-A"
```

需要添加`-A`是因为这台生产服务器藏在另外一台代理服务器后面。需要使用SSH agent forwarding多机器转发的功能。

四、创建`.ansistrano/rollback.yml`

```yaml
---
- name: Rollback your-app-name
  hosts: all
  vars:
    ansistrano_deploy_to: "/app/folder/name/old_webapp"
  roles:
    - { role: ansistrano.rollback }
```

五、开始部署

```shell
ansible-playbook -i .ansistrano/hosts/production -e "ansistrano_git_branch=master" .ansistrano/deploy.yml
```

总结

文章里展示了如何从0到1为遗留项目构建自动部署脚本。希望能帮助到你。