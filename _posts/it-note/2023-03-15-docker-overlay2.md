---
layout: post
title: Docker use the OverlayFS storage driver
date: 2023-03-15 10:12:15
categories: 分布式
tags: docker 
excerpt: Docker and OverlayFS in practice
---

docker 镜像是基于分层存储 Union FS 设计的。Docker 目前支持的 Union FS 包括 overlay2, fuse-overlayfs, btrfs, zfs, aufs,overlay, devicemapper 和 vfs。

可以通过下面的命令查看底层的 Union FS 存储驱动是什么 

```sh
docker info  
 
Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
```

overlay 有两个版本，overlay 和 overlay2， overlay2 是为了解决 overlay 的inode 节点不够的问题。 它们的存储驱动都是 OverlayFS。

overlay2 对 Linux 内核的要求是 version 4.0 或者更高。

更改 docker 底层的存储驱动之前，要先把容器和镜像保存到 Docker Hub 或 私有仓库中，这个就不用重新创建它们了。 

# 如何配置 docker overlay2 

1、停止 docker 服务 

2、备份原有 docker 的文件 

3、修改配置文件 `/etc/docker/daemon.json`，如果不存在则创建它。json 格式要正确。 

4、启动 docker 服务。

5、查看是不是已经配置成功。 

```sh
# 1
sudo systemctl stop docker

# 2
cp -au /var/lib/docker /var/lib/docker.bk

# 3
vim /etc/docker/daemon.json

{
  "storage-driver": "overlay2"
}

# 4
sudo systemctl start docker

# 5
docker info
```

#  overlay2 如何工作 

在 linux 主机上， 有两个目录， overlay2 把它们呈现为单个目录， 这个些目录被称之为层， 这个过程被称之联合挂载。

OverlayFS 将低层的目录称为 `lowerdir`，将上层目录称为 `upperdir`。统一后视图是公开的目录，叫 `merged` 。

overlay2 驱动程序原生支持多达 128 个较低的 OverlayFS 层。这种功能为 Docker 的层次相关命令(如 Docker build 和 Docker commit ) 提供了更好的性能，并且消耗后备文件系统上更少的 inode。 

镜像和容器在硬盘上的层如下： 

```sh 
ll /var/lib/docker/overlay2
drwx--x--- 12 root root 4096 Apr 25 03:37 ./
drwx--x--- 13 root root 4096 Apr 25 03:37 ../
drwx--x---  4 root root 4096 Apr 24 19:17 2ffv7isdx76992p0e1xe63w8l/
drwx--x---  4 root root 4096 Apr 24 19:10 4bfc2416e4df45d9728d6b30e0cb7a24cc5e79d8ab706f7f4eca65b4201a7728/
drwx--x---  3 root root 4096 Apr 24 19:10 5a363a401329642d4513bde699089649aa9744d4a660087365a80386565f5b1a/
drwx--x---  4 root root 4096 Apr 24 19:10 7df40ac6a9d5a2e56fff9bbdd6fb325a333c4fb4dcd4313b19e395c287afdfdc/
drwx--x---  4 root root 4096 Apr 24 19:13 87167d5bffd332084fdade32412ae5c9f1ca0d3e2356c88066a218ed9e050ac0/
drwx--x---  4 root root 4096 Apr 24 19:13 ea0fd665b5d79524b26c619e43a250338369bd68414be01eb50b1fb613171b61/
drwx--x---  3 root root 4096 Apr 24 19:07 iqfgvzrdel3gio9wqulzrbz9f/
drwx--x---  3 root root 4096 Apr 24 19:07 ko1strpiy4zuq8gwxc9hokw04/
drwx------  2 root root 4096 Apr 24 19:17 l/
drwx--x---  4 root root 4096 Apr 24 19:19 ounx9o9ydt0wnbap3hvqcirp1/
```
`/var/lib/docker/overlay2` 这个目录下的文件由 docker 管理。不要去做操作。 

`l`目录包含缩短的层标识符作为符号链接。这些标识符用于避免达到 mount 命令参数的页大小限制。


```sh
ll /var/lib/docker/overlay2/l/
total 28
drwx------  2 root root 4096 Apr 24 19:17 ./
drwx--x--- 12 root root 4096 Apr 25 03:37 ../
lrwxrwxrwx  1 root root   33 Apr 24 19:07 7BQ7X72HRISR7ZCGHGLD6NPIZJ -> ../ko1strpiy4zuq8gwxc9hokw04/diff/
lrwxrwxrwx  1 root root   72 Apr 24 19:10 7NXG5WLGONVJKKJGYFSRZ23OL2 -> ../87167d5bffd332084fdade32412ae5c9f1ca0d3e2356c88066a218ed9e050ac0/diff/
lrwxrwxrwx  1 root root   72 Apr 24 19:10 BOFYG6VCTBOTXYIFTHQANEZUBI -> ../4bfc2416e4df45d9728d6b30e0cb7a24cc5e79d8ab706f7f4eca65b4201a7728/diff/
lrwxrwxrwx  1 root root   33 Apr 24 19:07 F3XG6EZ3PJYVVQJFBXM3DG4L5G -> ../iqfgvzrdel3gio9wqulzrbz9f/diff/
lrwxrwxrwx  1 root root   33 Apr 24 19:13 HSJTNS5KPTJ3IP6JB3H6JMZTET -> ../2ffv7isdx76992p0e1xe63w8l/diff/
lrwxrwxrwx  1 root root   72 Apr 24 19:10 L53KG6ASJKV6CLXCWKJVVJOCNL -> ../5a363a401329642d4513bde699089649aa9744d4a660087365a80386565f5b1a/diff/
lrwxrwxrwx  1 root root   72 Apr 24 19:13 UPCEH2S24CH6FMLEUIA7VNRCSN -> ../ea0fd665b5d79524b26c619e43a250338369bd68414be01eb50b1fb613171b61/diff/
lrwxrwxrwx  1 root root   72 Apr 24 19:10 VZROBWGSVSJPZNOX7SO663OQZO -> ../7df40ac6a9d5a2e56fff9bbdd6fb325a333c4fb4dcd4313b19e395c287afdfdc/diff/
lrwxrwxrwx  1 root root   33 Apr 24 19:17 Y73E6QE355DROK3VEWK7C5AVOG -> ../ounx9o9ydt0wnbap3hvqcirp1/diff/
```

----

1、[Docker and OverlayFS in practice](https://gdevillele.github.io/engine/userguide/storagedriver/overlayfs-driver/)

2、[Use the OverlayFS storage driver](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)