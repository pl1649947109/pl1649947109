---
title: 第六讲——docker容器技术
id: 6
date: 2019-11-14 20:00:00
tags: linux
comment: true
---

### 介绍

Docker是一个开源的应用容器引擎，基于Go语言并遵循Apache2.0协议开源。

**Docker作用**：

它可以让开发者打包他们的用用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上面，也可以实现虚拟化。

<!----more---->

**机制**：

容器完全使用沙箱机制，相互之间不会有任何接口，更为重要的是容器性能开销极低。

**Dockerde 应用场景**：

Web应用的自动化打包和发布区。

自动化测试和持续集成和发布。

在服务型环境中部署和调整数据库或者其他的后台应用。

从头编译或者扩展现有的openshift或者cloud   foundry平台来大家自己的Paas环境。

注释：paas（plaiform as a server）就是平台即服务的意思，这是一种把应用服务的运行（这里指的是云服务）和开发环境作为一种服务提供的商业模式。

 **Docker的优点**：

1，简化程序：开发者可以打包自己的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，就可以实现虚拟化。把么开发者就可以在docker中直接对自己的成果进行管理，非常的方便简洁。

2，避免选择恐惧症：就是说docker以来的环境应用等等，都可以使用后docker进行打包部署。

3，节省开支：一方面云计算的到来，硬件租用变的便宜。另一方面，Docker和云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题和改变了虚拟化的方式。

### Docker架构

Docker使用的是cs架构，使用远程的API来管理和创建Docker容器。

Docker容器通过Docker镜像来创建。

**容器和镜像的关系**：

就相当于面向对象中的类和对象之间的关系，容器就相当于一个对象；镜像就相当于一个类。

 

![img](file:///C:\Users\pl\AppData\Local\Temp\ksohtml38876\wps1.jpg)

Docker镜像(Images)：用于创建Docker容器的模板。

Docker容器（Container）：独立运行的一个或者一组应用。

Docker客户端（Client）：Docker客户端通过命令行或者其他工具使用Docker API与Docker的守护进程同信。

Docker主机（Host）：一个物理机或者虚拟的机器用于执行守护进程和容器。

Docker仓库（Registry）：Docker仓库用来保存镜像，也可以理解为代码仓库

Docker Machine:它是一个简化的Docker安装的命令行工具，通过一个简单的命令就可以在相应的平台上安装Docker。

**Docker容器使用**：

1.Docker命令：查看docker客户端的所有命令项。

2.Docker [指定命令] --help:深入了解指定的命令和方法

**运行一个web应用**：

Docker run -d -p training/webapp python app.py  #载入镜像

解释：-d:让容器运行在后台。 -p：将容器内部使用的网络端口映射到我们使用的主机上面。

**查看正在运行的web应用容器**：

Docker ps

**查看web应用程序日志**：

Docker logs [ID或者名字]

Docker logs -f bf087dd89dfdf

解释：-f：让docker logs像使用tail -f一样输出容器内部的标准输出

**查看web应用程序容器的进程**：

Docker top 容器名称

查看docker底层信息，返回的是json文件，记录这docker容器的配置和状态信息

Docker inspect 容器名称

**停止web应用容器**:

Docker  stop  容器名称

**重启web应用容器**：

Docker  start 容器名称

**移除web应用容器**：

Docker rm 容器名称

（移除时必须先停止）

**Docker镜像的使用**：

当运行容器时，使用的镜像如果在本地不存在，docker就会自动从docker镜像仓库中下载，默认是从docker Hub公共镜像源下载。

**列出镜像列表**：

Docker images

解释：在同一个仓库源可以有不同版本的同一镜像（软件或者环境），

因为同一个镜像的版本不一样，我们可以通过他们的Tag来表示运行哪一个镜像。

比如：docker run -t -i ubuntu:15.10 /bin/bash

Docker run -t -i ubuntu 14.27 /bin/bash  就是两个不同的容器

**下载新镜像**：

Docker pull 镜像名称

**查找镜像**：

Docker search 镜像名称

**运行镜像**：

Docker run 镜像名称

**更新镜像**：

Apt-get update

**退出容器**：

Exit

**构建镜像**：

Docker build  

**设置镜像标签**：

Docker tag xx

**Docker容器连接**：

网络端口映射：(默认TCP连接)

首先创建一个应用容器：

Docker run -d -p training/webapp python app.py

然后使用：

Docker ps来查看容器端口绑定到主机的什么端口

指定容器端口绑定到主机的端口：

Docker run -d -p 5000:5000 training/webapp python app.py

指定容器绑定的网址：

Docker run -d -p 0.0.0.0:5000:5000 training/webapp app.py

 

注释：端口映射并不是唯一把docker链接到另一个容器的方法，docker有一个链接系统允许将多个容器连接到一起，共享连接信息。

容器命名：当我们创建一个容器的时候，docker自动会对他进行命名，为了方便我们可以自己给他们命名：

Docker run -d -p --name 自定义容器名称 training/webapp python app.py

 

 

实例：

Docker安装php

Docker安装mysql

Docker命令大全：

**详细介绍：**

http://www.runoob.com/docker/docker-command-manual.html

下面的命令都可以在上面的网址中找到，并有中文文档的说明。

大纲：

**容器生命周期管理：**

Run

Start/stop/restart

Kill

Rm

Pause/unpause

Create

Exec

**容器操作：**

Ps

Inspect

Top

Attach

Events

Logs

Wait

Export

Port

**容器rootfs命令：**

Commit

Cp

Diff

**镜像仓库：**

Login

Pull

Push

Search

**本地镜像管理：**

Images

Rmi

Tag

Build

History

Save

Import

**其他：**

Info

version



Docker资源汇总：

### **Docker官方英文资源**

docker官网：[http://www.docker.com](http://www.docker.com/)

Docker Windows 入门：https://docs.docker.com/docker-for-windows/

Docker CE(社区版) Ubuntu：https://docs.docker.com/install/linux/docker-ce/ubuntu/

Docker mac 入门：https://docs.docker.com/docker-for-mac/

Docker 用户指引：https://docs.docker.com/config/daemon/

Docker 官方博客：http://blog.docker.com/

Docker Hub: https://hub.docker.com/

Docker开源： https://www.docker.com/open-source

### **Docker中文资源**

Docker中文网站：https://www.docker-cn.com/

Docker安装手册：https://docs.docker-cn.com/engine/installation/

### **Docker 国内镜像**

阿里云的加速器：https://help.aliyun.com/document_detail/60750.html

网易加速器：http://hub-mirror.c.163.com

官方中国加速器：https://registry.docker-cn.com

ustc的镜像：https://docker.mirrors.ustc.edu.cn

daocloud：https://www.daocloud.io/mirror#accelerator-doc（注册后使用）

 

 

 

 