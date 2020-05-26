---
title: "Linux系统中部署docker容器"
author: 胡承
date: 2020-05-26 20:35:3 +0800
CreateTime: 2020-05-26 20:35:3 +0800
categories: C# asp.net core docker
---

Asp.Net core支持了跨平台以后，我们不得不使用跨平台的方案做部署。通常服务器我们都会选择使用linux，那么部署在linux系统中，我们主流的云部署方案是使用docker容器技术。今天来给大家分享下，如何在CentOS系统中部署docker容器。

<!-- more -->

在CentOS中，常用的包管理器是`yum`,我们需要完成软件的安装离不开包管理器。  
`sudo`语法表示获取管理员权限，该方法不需要知道用户名和密码即可提权。

## 卸载旧版本（如果有）

$ sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

按下回车即可，就可以卸载掉旧版本。

## 首次安装

安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

安装docker前，我们需要指定docker的仓库地址，用来下载和更新docker。  
引用的仓库地址：https://download.docker.com/linux/centos/docker-ce.repo

```
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

安装最新版本

```
$ sudo yum install docker-ce docker-ce-cli containerd.io

```

安装指定版本

//查看docker版本信息，然后通过指定下面列举的版本号进行安装
```
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```
版本号规则：通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1。

```
$ sudo yum install docker-ce-18.09.1 docker-ce-cli-18.09.1 containerd.io

```

安装完成后，启动docker
```
$ sudo systemctl start docker
```
安装docker的方法，网上有很多资料，这里主要也是实践一下，并且将docker部署的整个流程给串起来。

docker已经安装完成了，在实际使用docker前，我们需要为docker准备一个私有仓库，这样我们做的镜像就可以放到我们自己的服务器里面了。

下一篇我们再来探讨，如何建立私有仓库。

