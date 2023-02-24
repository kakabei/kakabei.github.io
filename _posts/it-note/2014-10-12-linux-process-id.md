---
layout: post
title: linux 进程间关系
date: 2014-10-13 18:08:39
categories: Linux
tags: 系统编程
excerpt: linux process communication about process id
---

# 进程与进程组

1 每个进程都隶属于一个进程组，所以它有 `PID` 信息还有 `PGID` 。可以用` pid_t getpgid( pid_t pid)` ;

2 每个进程组都有一个首领进程，其 `PGID` 和 `PID` 相同。进程组将一直存在，直到其中所有进程都退出，或者加入到其他进程组。

3 `setpgid(pid_t pid, pid_t pgid); ` 将 `PID` 为 `pid` 的进程的 `PGID` 设置为 `pgid`。如果 `pid` 和 `pgid` 相同，则由 `pid` 指定的进程将被设置为进程首领。如果 `pid` 为 0，则表示设置当前进程的 `PGID` 为 `pgid`; 如果 `pgid` 为0， 则使用 `pid` 作为目标 `PGID`。

4 一个进程只能设置自己的或者其子进程的 `PGID`。并且当子进程调用 exec 系列函数后，我们也不能再在父进程中对它设置 `PGID`。 

# 会话

一些有关联的进程组开成一个会话。创建一个会话的函数。
```c
pid_t setsid(void);
```

这个函数不能由进程组的首领进程调用。否则将产生一个错误。对于非组首领的进程，调用该函数不仅创建新会话，而且有如下额外效果：

（1）调用进程成为会话的首领，此时该进程是新会话的唯一成员.

（2）新建一个进程组，其 `PGID` 就是调用进程的 `PID`,调用进程成为该组的首领。

（3）调用进程将甩开终端，（如果有的话）

 
