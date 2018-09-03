---
layout: post
title: linux corefile 设置
date: 2014-10-15 19:08:12
categories: Linux
tags: 系统编程 
excerpt: linux corefile dump
---

程序运行的过程中,可能会因为一些隐藏的bug导致崩溃,为了在出问题时，及时记录所在环境的情况，所以要设置core文件的产生。其实其本质就是把进程的内存保存到文件中去。

#### 1.core文件的生成开关和大小限制

1）使用ulimit -c命令可查看core文件的生成开关。若结果为0，则表示关闭了此功能，不会生成core文件。
2）使用ulimit -c filesize命令，可以限制core文件的大小（filesize的单位为kbyte）。

3）若ulimit -c unlimited，则表示core文件的大小不受限制。如果生成的信息超过此大小，将会被裁剪，最终生成一个不完整的core文件。在调试此core文件的时候，gdb会提示错误。

#### 2.core文件的名称和生成路径

若系统生成的core文件不带其他任何扩展名称，则全部命名为core。新的core文件生成将覆盖原来的core文件。

1）/proc/sys/kernel/core_uses_pid可以控制core文件的文件名中是否添加pid作为扩展。文件内容为1，表示添加pid作为扩展名，生成的core文件格式为core.xxxx；为0则表示生成的core文件同一命名为core。
可通过以下命令修改此文件：
echo "1" > /proc/sys/kernel/core_uses_pid

2）proc/sys/kernel/core_pattern可以控制core文件保存位置和文件名格式。
可通过以下命令修改此文件：
echo "/corefile/core-%e-%p-%t" > core_pattern，可以将core文件统一生成到/corefile目录下，产生的文件名为core-命令名-pid-时间戳
以下是参数列表:

```sh
%p - insert pid into filename 添加pid
%u - insert current uid into filename 添加当前uid
%g - insert current gid into filename 添加当前gid
%s - insert signal that caused the coredump into the filename 添加导致产生core的信号
%t - insert UNIX time that the coredump occurred into filename 添加core文件生成时的unix时间
%h - insert hostname where the coredump happened into filename 添加主机名
%e - insert coredumping executable name into filename 添加命令名
```

#### 3.core文件的查看

core文件需要使用gdb来查看。

```sh
gdb ./a.out
core-file core.xxxx
```

使用bt命令即可看到程序出错的地方。
以下两种命令方式具有相同的效果，但是在有些环境下不生效，所以推荐使用上面的命令。
1）gdb -core=core.xxxx

```sh
file ./a.out
bt
```

2）gdb -c core.xxxx

```sh
file ./a.out
bt
```
