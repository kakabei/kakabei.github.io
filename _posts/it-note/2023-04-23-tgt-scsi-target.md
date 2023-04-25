---
layout: post
title: TGT Linux SCSI target framework 
date: 2023-04-23 10:12:15
categories: storage
tags: tgt ceph 
excerpt:  TGT Linux SCSI target framework fro creating and maintaining SCSI Targets
---
 
TGT 是一种在用户态下的 SCSI target 框架。 用于 iSCSI 和 iSER 传输协议。 支持多种方式访问设备块。 

框架包含了用户态下的 deamon 和工具。 

源码在：[https://github.com/fujita/tgt](https://github.com/fujita/tgt)

当前，TGT 支持的传输协议有：

- 以太网网卡的 iSCSI  target 软件驱动程序
- Infiniband 和 RDMA 网卡的 iSER target 软件驱动程序

支持可以访问的本地存储有： 

- aio, the asynchronous I/O interface also known as libaio.
- rdwr, smc and mmc, synchronous I/O based on the pread() and pwrite() system calls.
- null, discards all data and reads zeroes.
- ssc, SCSI tape support.
- sg and bsg, SCSI pass-through.
- glfs, the GlusterFS network filesystem.
- rbd, Ceph's distributed-storage RADOS Block Device.
- sheepdog, a distributed object storage system.

工作中之所以用到 TGT， 就是为了访问 ceph 的块设备。 

# iSCSI 协议

SCSI (Small Computer System Interface,小型计算机系统接口) 用于主机与外部设备之间的连接。SCSI 协议是主机与磁盘通信的基本协议。它由SCSI 控制器进行数据操作,SCSI控制器相当于一个小型CPU,有自己的命令集和缓存 。

而 iSCSI 协议就是让 SCSI 可以在互联网上传输。 


# TGT 的安装

两种方式： 

1、源码编译
 
```sh 
git clone https://github.com/fujita/tgt.git

sudo apt install -y ceph-common librados-dev  librbd-dev   manpages-zh  xsltproc   docbook-xsl

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
# TGT 的程序框架

TGT 的一个作用就是它可以在用户下创建管理块设备。这样不用每一次代码的更新都要重新编译内核。

tgtamd 是 TGT的管理工具，它是通过 unix socket 和 tgt daemon 通信。 

tgt daemon （进程名 tgtd）负责和内核态下的 tgt core 模块通信。 

tgt core 通过 target 驱动程序和 ceph 块设备通信。

![](/assets/dfs/tgt-2023-04-24-19-16-06.png)


要和 ceph 块设备通信，系统要安装 ceph 的 librados。

即安装时要先 `apt install -y ceph-common librados-dev  librbd-dev`

和 ceph 集群通信的是 ceph librados。

ceph 集群要安全认证，所以相关的配置文件要拷贝到 `/etc/ceph/` 下。 

安装完 tgt 后。可以使用 rbd 命令管理  ceph 集群上的块设备。rbd 是通过 tgtd 和 ceph 集群交互的。     


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

iSCSI 协议可以设置访问权限的，可以通过 IP 来限制访问，也可以通过用户名/密码方式限制访问。

4、查看创建的 target 和 lun 

```sh
tgtadm --lld iscsi --mode target --op show
```

更多的命令可以用 `man tgtadm` 查询。 


-----

1、[RDMA 架构与实践](https://houmin.cc/posts/454a90d3/)

2、[Infiniband技术简介](https://zhuanlan.zhihu.com/p/336499148)

3、[TGT学习总结](https://zhuanlan.zhihu.com/p/137047153)

4、[tgt网关源码解读](https://blog.csdn.net/Morry_Chan/article/details/100891020)

5、[linux iscsi网络的三种工具tgt iscsi_tgt targetcli](https://blog.csdn.net/qqqqqq999999/article/details/72547970)