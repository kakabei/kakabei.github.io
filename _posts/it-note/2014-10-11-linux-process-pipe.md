---
layout: post
title: linux 进程间通信-管道
date: 2014-10-11 21:20:33
categories: Linux
tags: 系统编程
excerpt: llinux process communication about pipe
---

### 一 管道的局限性

管道有两个局限性：（1）他是半双工（即数据只能在一个方向上流动）。（2）它只能在具有公共祖先的进程之间使用。一个管道由一个进程创建，然后该 进程调用fork，此后父子进程之间就可该管道。

### 二 管道的创建

用函数pipe创建：

```c
#include<unistd.h>
int pipe(int files[2]);
```

参数：filedes返回两个文件描述符：filedes[0] 为从管道读而打开，filedes[1]为管道写而打开。filedes[1]的输出是filedes[0]的输入。

### 三 管道原理图

如下这张图可以说明管道的原理，即一个父进程创建一个子进程后，父进程打开写管道，子进程打开读管道。

![](/assets/linux/linux-process-pipe-1.png) 

### 四 管道只有一端时的情况

当管道的一端被关闭后，下列规则起作用：
(1)  当读一个写端已被关闭的管道时，在所有数据都被读取后， read返回0，以指示达到了文件结束处（从技术方面考虑，管道的写端还有进程时，就不会产生文件的结束。可以复制一个管道的描述符，使得有多个进程具有写打开文件描述符。但是，通常一个管道只有一个读进程，一个写进程。下一节介绍F I F O时，我们会看到对于一个单一的FIFO常常有多个写进程）。
(2)  如果写一个读端已被关闭的管道，则产生信号 SIGPIPE。如果忽略该信号或者捕捉该信号并从其处理程序返回，则 write 出错返回，errno设置为EPIPE。
(3) 在写管道时，常数 PIPE_BUF 规定了内核中管道缓存器的大小。如果对管道进行 write调用，而且要求写的字节数小于等于 PIPE_BUF，则此操作不会与其他进程对同一管道（或 FIFO )的write操作穿插进行。但是，若有多个进程同时写一个管道（或 FIFO） ，而且某个或某些进程要求写的字节数超过 PIPE_BUF字节数，则数据可能会与其他写操作的数据相穿插。

### 五 例子
```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#define  MAXLINE   (2014)
int main(void)
{
    int     n, fd[2];
    pid_t   pid;
    char    line[MAXLINE];
    if (pipe(fd) < 0)
        printf("pipe error");
    if((pid = fork()) < 0)
        printf("fork error");
    else if (pid > 0) 
    {             /* parent */
        close(fd[0]);/* close read */
        printf ("the process pid %d\n", getpid());
        printf ("the process write to pipe : hello world!\n");
        write(fd[1], "hello world\n", 12);
     } 
     else
     {                                /* child */
        close(fd[1]); /* close write */
        printf("the process pid %d\n", getpid());
        n = read(fd[0], line, MAXLINE);
        write(STDOUT_FILENO, line, n);
     }
     exit(0);
}
```








