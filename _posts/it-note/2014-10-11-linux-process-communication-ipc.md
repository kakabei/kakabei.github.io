---
layout: post
title: linux 进程间通信-XSI IPC
date: 2014-10-11 18:10:30
categories: Linux
tags: 系统编程
excerpt: linux process communication about XSI IPC
---

### 一 什么是XSI IPC
有三种 IPC我们称作XSI IPC，即消息队列、信号量以及共享存储器（共享内存），它们之间有很多相似之处。

### 二 标识符和键
每个内核中的 IPC结构（消息队列、信号量或共享内存）都用一个非负整数的标识符加以引用。无论何时创建IPC结构（调用 msgget、semget 或shmget）,都应指定一个关键字（key），关键字的数据类型由系统规定为 key_t，通常在头文件<sys/types.h>中被规定为长整型。关键字由内核变换成标识符。
  key的取法有两种：ftok和IPC_PRIVATE，（网上看到的）

### 三 两个进程（服务和客户）使用同一个IPC结构的方法
(1) 服务器进程可以指定键IPC_PRIVATE创建一个新的IPC结构，将返回的标识符存放在某处（如一个文件）以便和客户进程取用。也可用父子进程的方式传递。
(2) 在一个公用头文件 中定义 一个客户进程和服务器进程都认可的键。然后服务器进程指定此键创建一个新的IPC结构。
(3) 客户进程和服务器进程认同一个路径和项目ID（项目ID是０～２５５间的字符值），接着调用函数ftok将这两个值变换一个键。然后使用此键。
```c
#inlcude <sys/ipc.h>
key_t ftok(const char *path, int id);
```
path 参数必须引用一个现存文件，当产键时，保使用id参数的低8位。如果使用同一项目ID，那么对于不同文件 的两个路径可能产相同的键。

### 四 权限结构
XSI IPC为每一个IPC结构设置了一个 ipc _ perm结构。该结构规定了许可权和所有者。主要成员有：
```c
struct ipc_perm {
uid_t uid ;  /* owner's effective user id */
gid_t gid ;  /* owner's effective group id */
uid_t cuid;  /* creator's effective user id */
gid_t cgid ; /* creator's effective group id */
mode_t mode; /* access modes */
ulong seq ;  /* slot usage sequence number */
key_t key;   /* key */
}
```

详细的见<sys/ipc.h>在linux的源码中可以找到。
在创建IPC结构时，对所有字段都赋初值。以后，可以调用msgctl、semctl或shmctl修改uid、gid和mode字段。为了改变这些值，调用进程必须是IPC结构的创建者或超级用户。

### 五 Linux中，与IPC相关的命令包括：ipcs、ipcrm（释放IPC）
IPCS命令是Linux下显示进程间通信设施状态的工具。我们知道，系统进行进程间通信（IPC）的时候，可用的方式包括信号量、共享内存、消息队列、管道、信号（signal）、套接字等形式。使用IPCS可以查看共享内存、信号量、消息队列的状态。
例如在CentOS6.0上执行ipcs
具体的用法总结如下：
```sh
#1、显示所有的IPC设施
 ipcs -a
#2、显示所有的消息队列Message Queue
 ipcs -q
#3、显示所有的信号量
 ipcs -s
#4、显示所有的共享内存
 ipcs -m
#5、显示IPC设施的详细信息
 ipcs -q -i id
#id 对应shmid、semid、msgid等。-q对应设施的类型（队列），查看信号量详细情况使用-s，查看共享内存使用-m。
#6、显示IPC设施的限制大小
 ipcs -m -l
#-m对应设施类型，可选参数包括-q、-m、-s。
#7、显示IPC设施的权限关系
 ipcs -c
 ipcs -m -c
 ipcs -q -c
 ipcs -s -c
#8、显示最近访问过IPC设施的进程ID。
 ipcs -p
 ipcs -m -p
 ipcs -q -p
#9、显示IPC设施的最后操作时间
 ipcs -t
 ipcs -q -t
 ipcs -m -t
 ipcs -s -t
#10、显示IPC设施的当前状态
 ipcs -u
 ```
Linux上的ipcs命令，不支持UNIX上的-b、-o指令，同样UNIX中不支持-l、-u指令，所以在编写跨平台的脚本时，需要注意这个问题。