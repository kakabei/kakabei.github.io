---
layout: post
title: TGT Linux SCSI target framework 
date: 2023-04-23 10:12:15
categories: 系统设计
tags: tgt ceph 
excerpt: 
---

支持 SCSI target 驱动协议， 如:iSCSI、Fibre Channel、SRP 等。 创建和维护的 SCSI target 的框架。 

tgt 的目的是把传统的 SCSI target 协议驱动的创建和维护从内核态转至用户态。避免每一次修改代码都需要重新编译内核。同时在用户态下，开发者可以使用多种第三方库以及调试工具等，减轻了开发人员的负担。

源码在：[https://github.com/fujita/tgt](https://github.com/fujita/tgt)

# TGT 的安装

两种方式： 

1、源码编译
 
```sh 
git clone https://github.com/fujita/tgt.git

sudo apt install -y lceph-common librados-dev  librbd-dev   manpages-zh  xsltproc   docbook-xsl

cd tgt

make

make install
```
安装后文件的目录

```sh 
install -d -m 755 /usr/sbin
install -m 755 tgtd tgtadm tgtimg /usr/sbin
install -d -m 755 /usr/lib/tgt/backing-store
install -m 755 bs_rbd.so /usr/lib/tgt/backing-store
```

2、 apt 直接安装 

```sh
sudo apt install tgt-rbd
```

# TGT 常用的命令


1、创建 target

```sh
tgtadm --lld iscsi --mode target --op new --tid 1 --targetname iqn.2009-02.com.example:for.all
```
targetname 是我们要创建的 target 的名字，其采用IQN 命名格式：iqn.年-月.反向域名:任何唯一的名字

2、创建 lun

```sh
tgtadm --lld iscsi --mode logicalunit --op new --tid --lun 1 --backing-store /mnt/disk.img
```

backing-store 代表我们后端存储的文件名，我们不是真正的磁盘；要模拟磁盘的数据保存功能，可以将数据保持在这个文件里。

3、设置访问权限（此处我们允许所有IP访问）

```sh
tgtadm --lld iscsi --mode target --op bind --tid 1 -I ALL
```

iSCSI协议可以设置访问权限的，可以通过IP来限制访问，也可以通过用户名/密码方式限制访问。

4、查看创建的target和lun 

```sh
tgtadm --lld iscsi --mode target --op show
```

更多的命令可以用 `man tgtadm` 查询

# TGT 的程序框架

![](/assets/dfs/tgt-2023-04-24-19-16-06.png)

![](assets/dfs/tgt-2023-04-24-19-20-03.png)