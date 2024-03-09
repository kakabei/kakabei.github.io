---
layout: post
title: linux 进程间通信-XSI IPC
date: 2014-10-11 18:10:30
categories: Linux
tags: 系统编程
excerpt: linux process communication about XSI IPC
---

# 一 什么是 XSI IPC


XSI IPC (X/Open System Interface Interprocess Communication) 是一组进程间通信的API标准，由 X/Open 组织定义并提供。XSI IPC 包括三种进程间通信方式：**共享内存**、**消息队列**和**信号量**。这三种通信方式都具有跨平台和可移植性的特点，能够在多种操作系统和硬件平台上使用。
# 二 标识符和键

每个内核中的 IPC 结构（消息队列、信号量或共享内存）都用一个非负整数的标识符加以引用。无论何时创建 IPC 结构（调用 msgget、semget 或shmget）,都应指定一个关键字（key），关键字的数据类型由系统规定为 key_t，通常在头文件`<sys/types.h>`中被规定为长整型。关键字由内核变换成标识符。

key的取法有两种：ftok 和 IPC_PRIVATE。 

# 三 两个进程（服务和客户）使用同一个 IPC 结构的方法

1、服务器进程可以指定键 IPC_PRIVATE 创建一个新的 IPC 结构，将返回的标识符存放在某处（如一个文件）以便和客户进程取用。也可用父子进程的方式传递。

2、在一个公用头文件 中定义 一个客户进程和服务器进程都认可的键。然后服务器进程指定此键创建一个新的 IPC 结构。

3、客户进程和服务器进程认同一个路径和项目ID（项目ID是０～２５５间的字符值），接着调用函数 `ftok` 将这两个值变换一个键。然后使用此键。

`ftok` 函数的用法:

```c++
key_t ftok(const char *pathname, int proj_id);
```

根据文件路径名和项目ID生成一个唯一的IPC键。它通常用于创建System V IPC机制中的消息队列、共享内存和信号量等对象。

函数参数说明：

- pathname：文件路径名，用于生成IPC键的基础信息。如果不想使用任何文件来生成IPC键，可以将该参数设置为任意非空字符串，通常选择使用当前程序的路径。

- proj_id：项目ID，是一个整数值，用于区分同一文件不同IPC对象之间的区别。

返回值说明：

ftok函数返回一个唯一的IPC键。如果出现错误，将返回-1。

```c++
#include <sys/ipc.h>
#include <sys/msg.h>

#define MSGKEY_PATH "/tmp/msgkey_path"

int main() {
    key_t msgkey = ftok(MSGKEY_PATH, 'm');
    int msgid = msgget(msgkey, 0666 | IPC_CREAT);
    // 进行消息队列的操作...
    return 0;
}
```

`ftok` 函数生成唯一的 IPC 键msgkey，并通过该键创建了一个消息队列 msgget。该 IPC 键由 MSGKEY_PATH 和 'm' 共同生成，用于区分不同的 IPC 对象。


# 四 权限结构

`XSI IPC` 为每一个 IPC 结构设置了一个 `ipc_perm` 结构。该结构规定了许可权和所有者。主要成员有：
```c
struct ipc_perm {
    key_t key;          /* 键值 */
    uid_t uid;          /* 所有者的用户ID */
    gid_t gid;          /* 所有者的组ID */
    uid_t cuid;         /* 创建者的用户ID */
    gid_t cgid;         /* 创建者的组ID */
    mode_t mode;        /* 访问权限 */
    unsigned short seq; /* 序列号 */
};

```
详细的见`<sys/ipc.h>`在 linux 的源码中可以找到。

在创建IPC结构时，对所有字段都赋初值。以后，可以调用 msgctl、semctl 或 shmctl 修改 uid、gid 和 mode 字段。为了改变这些值，调用进程必须是IPC结构的创建者或超级用户。

```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/sem.h>

#define SEMKEY_PATH "/tmp/semkey_path"

int main() {
    // 生成唯一IPC键
    key_t semkey = ftok(SEMKEY_PATH, 's');
    // 创建IPC信号量，设置ipc_perm结构
    int semid = semget(semkey, 1, 0666 | IPC_CREAT);
    if (semid == -1) {
        perror("semget error");
        exit(1);
    }

    // 获取ipc_perm结构
    struct semid_ds sem_ds;
    semctl(semid, 0, IPC_STAT, &sem_ds);

    // 设置ipc_perm结构
    sem_ds.sem_perm.uid = getuid();
    sem_ds.sem_perm.gid = getgid();
    sem_ds.sem_perm.mode = 0600;

    // 更新ipc_perm结构
    if (semctl(semid, 0, IPC_SET, &sem_ds) == -1) {
        perror("semctl error");
        exit(1);
    }

    printf("Semaphore created successfully.\n");

    return 0;
}
```

`ipc_perm` 结构和 `struct ipc_perm` 结构是同一个数据类型的别名。`ipc_perm` 是`<sys/ipc.h>`头文件中定义的一个宏，它被定义为 `struct ipc_perm` 类型。这样定义的好处是可以简化代码的书写，同时也能使代码更加易读。

```c++
#include <sys/ipc.h>

key_t key = ftok("/tmp/semkey_path", 's');
int semid = semget(key, 1, 0666 | IPC_CREAT);
struct ipc_perm perm = { getuid(), getgid(), 0600, 0 };
semctl(semid, 0, IPC_SET, &perm);
```
声明了一个ipc_perm类型的变量perm，并将其成员初始化为进程的用户ID、组ID以及访问权限。在调用semctl函数时，我们将该结构作为参数传递给函数，以设置IPC信号量的权限和所有权。
# 五 Linux中，与IPC相关的命令包括：ipcs、ipcrm（释放IPC）

IPCS 命令是 Linux 下显示进程间通信设施状态的工具。我们知道，系统进行进程间通信（IPC）的时候，可用的方式包括信号量、共享内存、消息队列、管道、信号（signal）、套接字等形式。使用IPCS可以查看共享内存、信号量、消息队列的状态。

例如在 CentOS6.0 上执行 ipcs

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
Linux上的ipcs命令，不支持 UNIX 上的 -b、-o 指令，同样 UNIX 中不支持 -l、-u 指令，所以在编写跨平台的脚本时，需要注意这个问题。