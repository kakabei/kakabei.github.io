---
layout: post
title: docker image container repository 
date: 2023-03-13 10:12:15
categories: 分布式
tags: docker 
excerpt: docker develop faster, run anywhere
---

Docker 是一种操作系统层面上的虚拟技术，基于 Linux 内核的 cgroup namespace union fs 技术对进程进行封装隔离。 

隔离后的进行独立于宿主和其他进程，所以也称为容器。

Docker 是 golang 进行开发实现的。

Docker 框架：

![](/assets/docker/docker-2023-04-25-19-42-49.png)


和传统的虚拟机技术的区别：

传统的虚拟机技术是要虚拟出一套硬件，在这个硬件上运行一个操作系统，然后进程在这个操作系统上运行。

而 Docker 技术是让进程直接运用宿主机器上的内核，容器没有自己的内核。其实就是所有容器共有宿主的内核。 


Docker 的三个基本概念,  是 Docker 的整个生命周期

- 镜像（Image）
- 容器（Container）
- 仓库（Repository）


# 镜像

镜像是一个特殊的文件系统，除必要的程序、资源、配置外，不包含其他动态数据。

镜像是基于分层存储 Union FS设计的。它并非一个像ISO的打包文件，而是由多层文件系统联合组成。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

分层存储的特征还使得镜像的复用、定制变的更为容易。可以充分的利用之前的基础进一步添加新的层，以定制自己所需的内容，构建新的镜像。

# 容器

如果我个把镜像当作一个静态的文件，那么容器就是这个静态文件的运行状态，可以类比可执行文件和跑起的进程的场景。

容器是镜像的运行的实体，其实本质就是一个进程，可以创建、启动、停止、删除、暂停。

容器拥有自己的命名空间、root 文件系统、网络配置、进程空间、用户Id。容器内的进程是运行在一个隔离空间。除了内核其他的都被隔离开。 

容器运行时，以镜像为基础层，在其上创建一个当前容器的存储层，这个为容器运行而准备的。叫“容器存储层”。容器消亡时，这个容器存储层也同样会消亡。 

所以，不应该把数据写到 容器的存储层上。而是应该使用数据卷。即把宿主的一个目录映射到容器里。这样直接读写在宿主的存储上，性能会更优。 数据卷不会因为容器的消亡而消失。 

# 仓库

为了能把一个镜像跑在不同的服务器宿主上，我们需要一个集中存储分发镜像的服务，这就是镜像仓库 Docker Registry。

Docker Registry 可以有多个仓库，仓库可以包含多个标签。每个标签对应一个镜像。

如下： 仓库名：ubuntu 标签：16.04、18.04、latest。
```txt 
ubuntu:16.04
ubuntu:18.04
ubuntu:latest
```
Docker Registry 有公开的服务，也有私有的服务。 

公开的有： 
- [Docker Hub](https://hub.docker.com/)
- [Red Hat Quay.io](https://quay.io/repository/)


也可以自己搭建私有的 Docker Registry。可以用官方提供的工具 [docker-registry](https://docs.docker.com/registry/) 搭建私有的 Docker Registry。

# 私有仓库

pass





