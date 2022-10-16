---
layout: post
title: linux 进程间关系
date: 2014-10-13 18:08:39
categories: Linux
tags: 系统编程
excerpt: linux process communication about process id
---


1 每个进程都隶属于一个进程组，所以它有PID信息还有PGID。可以用pid_t getpgid( pid_t pid);
2 每个进程组都有一个首领进程，其PGID和PID相同。进程组将一直存在，直到其中所有进程都退出，或者加入到其他进程组。
3 setpgid(pid_t pid, pid_t pgid); 将PID为pid的进程的PGID设置为pgid。如果pid和pgid相同，则由pid指定的里程将被设置为进程首领。如果pid为0，则表示设置当前进程的PGID为pgid; 如果pgid为0.则使用pid作为目标PGID。
4 一个进程只能设置自己的或者其子进程的PGID。并且当子进程调用 exec系列函数后，我们也不能再在父进程中对它设置PGID。 

会话
一些有关联的进程组开成一个会话。创建一个会话的函数
pid_t setsid(void);

这个函数不能由进程组的首领进程调用。否则将产生一个错误。对于非组首领的进程，调用该函数不仅创建新会话，而且有如下额外效果：
（1）调用进程成为会话的首领，此时该进程是新会话的唯一成员
（2）新建一个进程组，其PGID就是调用进程的PID,调用进程成为该组的首领。
（3）调用进程将甩开终端，（如果有的话）

 
