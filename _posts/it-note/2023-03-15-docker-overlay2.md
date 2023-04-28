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

```sh
sudo systemctl stop docker
```

2、备份原有 docker 的文件 

```sh
cp -au /var/lib/docker /var/lib/docker.bk
```

3、修改配置文件 `/etc/docker/daemon.json`，如果不存在则创建它。json 格式要正确。 

```sh
vim /etc/docker/daemon.json

{
  "storage-driver": "overlay2"
}
```
4、启动 docker 服务。

```sh
sudo systemctl start docker
```

5、查看是不是已经配置成功。 

```sh
docker info
```
如果 `Storage Driver: overlay2` 则说明修改成功。
#  overlay2 如何工作 

在 linux 主机上， 有两个目录， overlay2 把它们呈现为单个目录， 这个些目录被称之为层， 这个过程被称之联合挂载。

OverlayFS 将低层的目录称为 `lowerdir`，将上层目录称为 `upperdir`。统一后视图是公开的目录，叫 `merged` 。

overlay2 原生支持多达 128 个 OverlayFS 的`lowerdir`层。这种功能为 Docker 的层次相关命令(如 Docker build 和 Docker commit ) 提供了更好的性能，并且消耗后备文件系统上更少的 inode。 

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
The lowest layer contains a file called link, which contains the name of the shortened identifier, and a directory called diff which contains the layer’s contents.

最低层包含的一个叫 link的文件，它包含缩短标识符的名称，以及一个叫 diff 的目录，该目录包含了该层的内容。

```sh
ll /var/lib/docker/overlay2/4bfc2416e4df45d9728d6b30e0cb7a24cc5e79d8ab706f7f4eca65b4201a7728
total 24
drwx--x---  4 root root 4096 Apr 24 19:10 ./
drwx--x--- 12 root root 4096 Apr 25 03:37 ../
-rw-------  1 root root    0 Apr 24 19:10 committed
drwxr-xr-x  7 root root 4096 Apr 24 19:10 diff/
-rw-r--r--  1 root root   26 Apr 24 19:10 link
-rw-r--r--  1 root root   28 Apr 24 19:10 lower
drwx------  2 root root 4096 Apr 24 19:10 work/
root@LAPTOP-0ODB1KTF:/home/kane/cloudsky#
```
可分别查看 committed、diff、link、lower、work的内容。 
# overlay 如何工作 

在 linux 主机上， 有两个目录， overlay2 把它们呈现为单个目录， 这个些目录被称之为层， 这个过程被称之联合挂载。

OverlayFS 将低层的目录称为 `lowerdir`，将上层目录称为 `upperdir`。统一后视图是公开的目录，叫 `merged` 。

下图显示 docker 的镜像和容器的是如何分层的。 镜像是 lowerdir， 容器是upperdir 。联合的视图通过目录 merged 呈现出来。就是容器的实际挂载点。
这个展示了 docker 结构和 overlay FS 的映射关系 。

![](/assets/docker/docker-overlya2-2023-04-28-22-05-30.png)

当镜像和容器出现相同的文件时，以容器的文件为主，它会屏蔽掉镜像中的文件。

overlay 只有两层，无法直接实现 OverlayFS 的多层。所以用另外一种方式去实现， 即用硬链接的方式与  `lowerdir` 共享数据。硬链接的过度使用导致 inode 节点不够。这是 overlay 遗留下来的缺陷。 

创建容器时，overlay 当表示容器的那一层和镜像的那一层联合在一起。形成一个新的目录 。镜像的 `lowerdir` 层是只读，而容器的 `upperdir` 是可写的。


# 容器是如何在 overlay 或 overlay2 进行的读写的

## 读文件 

有三种场景，容器会通过 overlay 打开并读出文件。

- **文件不存在容器层:** 如果容器打开一个文件，它不存在容器这一层，即 `upperdir`,它就会去 镜像那一层，即 `lowerdir`读取。通过 overhead 性能消耗很低。 
- **文件存在容器层:** 如果容器打开一个文件，它刚好存在这一层 `upperdir`，直接读取即可。 
- **文件存在容器层和镜像层:**。如果容器打开的文件 `upperdir` 和 `lowerdir` 层都存在，则 `upperdir`层的文件会隐藏掉`lowerdir`的文件，即只会读到`upperdir`层的文件。 

## 修改文件和目录

有以下几种场景，容器被修改
 
- **第一次写文件:** 容器第一次写文件，这个文件存在于镜像层（ `lowerdir` 层）但不存在于容器层（ `upperdir` 层）。overlay 或 overlay2 会执行一个 `copy_up` 的方法。把文件从 `lowerdir` 层拷贝到  `upperdir` 层。 容器随把写入的更新到新拷贝到`upperdir` 层的文件里。 
然后，  OverlayFS是工作在到文件层级而不是块层级的，这就意味着 OverlayFS 的 `copy_up` 方法要拷贝整个文件。 如果一个文件很大，但改动很小，同样要拷贝整个文件。这个很明显会对容器写的性能造成很大的影响。 因此，以下两点很值得注意：

  - `copy_up` 方法的操作只发生在第一次写入文件时的情况，后继的写入同个文件则是在已被拷贝到容器层的文件副本进行的。
  - OverlayFS 只有两次，所以它的性能比 AUFS 好，后者需要搜索很多层的镜像的文件时可能会有些性能上损耗。overlay 和 overlay2 都有这个优点。Overlayfs2 在初始读取时的性能略低于overlayfs，因为它必须遍历更多的层，但它会缓存结果，所以这只是一个很小的损失。 

- **删除文件或目录：**
  
  - 当容器里的一个文件被删除时，会在容器层（`upperdir` 层）写入一个 whiteout 文件。 镜像层（ `lowerdir` 层）中的文件不会被删除，因为镜像层（ `lowerdir` 层）中只读。而 whiteout 文件让它在容器里失效，成了已被删除的文件。 
  - 当容器里的一个目录被删除时，会在容器层 （`upperdir` 层） 创建一个  opaque 目录。和 whiteout 文件方式一样，让阻止了 镜像层（ `lowerdir` 层）中的目录生效，  镜像层（ `lowerdir` 层）中的目录是依然存在的。 

- **重命名目录:** 只有当源目录和目的目录都在容器层（`upperdir` 层），调用 `rename(2)` 重命名目录者才可以成功，否则，它返回 EXDEV 错误(“不允许跨设备链接”) 应用层应该处理好  EXDEV 错误。 


# OverlayFS and Docker 的性能 


overlay2 和 overlay 的性能比 aufs 和 devicemapper 都要高出很多。 在某些情况下，overlay2 也可能比 btrfs 表现得更好。但是，也要注意以下细节： 


- **页缓存** OverlayFS 支持页缓存共存。 访问同一文件的多个容器共享该文件的单个页面缓存项。这使得 overlay 和 overlay2 在内存方面效率很高，对于 PaaS 等高密度用例来说是一个很好的选择。
- **copy_up 方法**  与 AUFS 一样，OverlayFS 在容器第一次写入文件时会执行复制操作。这会降低写的性能，特别是对于大文件。但是，一旦文件被复制，随后对该文件的所有写操作都发生在容器层（`upperdir` 层） ，而不需要进一步的复制操作。

OverlayFS 的 `copy_up` 操作比使用 AUFS 的相同操作要快，因为 AUFS 比 OverlayFS 支持更多的层。 如果搜索许多 AUFS 层，可能会导致更大的延迟。Overlay2 也支持多层，但是通过缓存减轻了性能损失。

- **Inode 限制** 使用 overlay 会导致inode消耗过多。在 Docker 主机上存在大量镜像和容器时 inode 消耗更利害。而增加文件系统可用 inode 数量的唯一方法是重新格式化。为了避免遇到这个问题，强烈建议尽可能使用 overlay2。 
# OverlayFS 存在的局限

总结下 OverlayFS 和其他文件系统不兼容的方面。 

- **open(2):** OverlayFS 只实现 POSIX 标准的一个子集。这就导致一些  OverlayFS 操作违反了 POSIX 标准， 如 `copy_up` 操作。如程序的调用
```c++
fd1 = open("foo", O_RDONLY) 
fd2 = open("foo", O_RDWR)
```
我们的理解这是同个文件，但在第二个`open `发生了 `copy_up` 操作，描述符指向不同的文件。  fd1 指向了镜像层（ `lowerdir`层）因为它是只读。而fd2 指向的是容器层（`upperdir` 层）。导致这个问题的原因就是触发了`copy_up` 操作。`open(2)` 后的所有的操作无论是读和写都发生在容器层（`upperdir` 层）。

- **rename(2):** OverlayFS 没有全部支持 `rename(2)`系统调用，应用上考虑检测是否因为失败和回滚策略，

----

1、[Docker and OverlayFS in practice](https://gdevillele.github.io/engine/userguide/storagedriver/overlayfs-driver/)

2、[Use the OverlayFS storage driver](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)