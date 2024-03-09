---
layout: post
title: linux 进程间通信-消息队列
date: 2014-10-12 18:08:39
categories: Linux
tags: 系统编程
excerpt: linux process communication about message queue
---

# 消息队列的介绍

消息队列提供了一种从一个进程向另一个进程发送一个数据块的方法。

每个数据块都被认为含有一个类型，接收进程可以独立地接收含有不同类型的数据结构。

我们可以通过发送消息来避免命名管道的同步和阻塞问题。

Linux用宏 MSGMAX 和 MSGMNB 来限制一条消息的最大长度和一个队列的最大长度。

影响消息队列的系统限制

- **MSGMAX**   可发送是最长消息的长度                             2048
- **MSGMNB**   特定队列的最大字节长度（亦即队列中所有消息之和）        4096
- **MSGMNI**   系统中最大消息列数                                 50
- **MSGTOL**   系统中最大消息数                                   50

# 消息队列的实现

主要从如下几个函数实现（就四个动作，创建、发送、接收、删除）

## 1. 创建和访问消息队列原型：

```c
int msgget(key_t, key, int msgflg);
```

* 程序必须提供一个键来命名这个消息队列。`msgflg` 是一个权限标志，表示消息队列的访问权限，它与文件的访问权限一样。
* `msgflg` 可以与 `IPCCREAT` 做或操作，表示当 key 所命名的消息队列不存在时创建一个消息队列，如果 key 所命名的消息队列存在时，`IPC_CREAT` 标志会被忽略，而只返回一个标识符。
* 函数返回一个以 key 命名的消息队列的标识符（非零整数），失败时返回-1.
 
## 2. 把消息添加到消息队列原型：

```c
int msgsend(int msgid, const void *msg_ptr, size_t msg_sz, int msgflg);
```

* **msgid** 是由 `msgget` 函数返回的消息队列标识符。
* **msg_ptr** 是一个指向准备发送消息的指针，但是消息的数据结构却有一定的要求，指针 `msg_ptr` 所指向的消息结构一定要是以一个长整型成员变量开始的结构体，接收函数将用这个成员来确定消息的类型。所以消息结构要定义成这样：

```c
struct my_message{
    long int message_type;
    /*  The data you wish to transfer */
};
```
* **msg_sz** 是 **msg_ptr** 指向的消息的长度，注意是消息的长度，而不是整个结构体的长度，也就是说 **msg_sz** 是不包括长整型消息类型成员变量的长度。
* **msgflg** 用于控制当前消息队列满或队列消息到达系统范围的限制时将要发生的事情。
* 调用成功，消息数据的一分副本将被放到消息队列中，并返回0，失败时返回-1.

### 3. 从一个消息队列获取消息原型:

```c
int msgrcv(int msgid, void *msg_ptr, size_t msg_st, long int msgtype, int msgflg);
```

* **msgid** **msg_ptr**, **msg_st** 的作用也函数 `msgsnd` 函数的一样。
* **msgtype** 可以实现一种简单的接收优先级。如果 **msgtype** 为 0，就获取队列中的第一个消息。如果它的值大于零，将获取具有相同消息类型的第一个信息。如果它小于零，就获取类型等于或小于 msgtype 的绝对值的第一个消息。
* **msgflg** 用于控制当队列中没有相应类型的消息可以接收时将发生的事情。
* 调用成功时，该函数返回放到接收缓存区中的字节数，消息被复制到由 `msg_ptr` 指向的用户分配的缓存区中，然后删除消息队列中的对应消息。失败时返回-1。

### 4.控制消息队列原型：

```c
int msgctl(int msgid, int command, struct msgid_ds *buf);
```

* **command** 是将要采取的动作，它可以取3个值，
  * IPC_STAT：把msgid_ds结构中的数据设置为消息队列的当前关联值，即用消息队列的当前关联值覆盖 `msgid_ds` 的值。
  * IPC_SET：如果进程有足够的权限，就把消息列队的当前关联值设置为 `msgid_ds` 结构中给出的值。
  * IPC_RMID：删除消息队列。
* **buf** 是指向 `msgid_ds` 结构的指针，它指向消息队列模式和访问权限的结构。`msgid_ds` 结构至少包括以下成员：

```c
struct msgid_ds
{
    uid_t shm_perm.uid;
    uid_t shm_perm.gid;
    mode_t shm_perm.mode;
};
```

# 例子

```c
#include <unistd.h>  
#include <stdlib.h>  
#include <stdio.h>  
#include <string.h>  
#include <errno.h>  
#include <sys/msg.h>  
      
struct msg_st  
{  
    long int msg_type;  
    char text[BUFSIZ];  
};  
      
int main()  
{  
    int running = 1;  
    int msgid = -1;  
    struct msg_st data;  
    long int msgtype = 0; //注意1  
      
    //建立消息队列  
    msgid = msgget((key_t)1234, 0666 | IPC_CREAT);  
    if(msgid == -1)  
    {  
        fprintf(stderr, "msgget failed with error: %dn", errno);  
        exit(EXIT_FAILURE);  
    }  
    //从队列中获取消息，直到遇到end消息为止  
    while(running)  
    {  
        if(msgrcv(msgid, (void*)&data, BUFSIZ, msgtype, 0) == -1)  
        {  
            fprintf(stderr, "msgrcv failed with errno: %dn", errno);  
            exit(EXIT_FAILURE);  
        }  
        printf("You wrote: %sn",data.text);  
        //遇到end结束  
        if(strncmp(data.text, "end", 3) == 0)  
            running = 0;  
    }  
    //删除消息队列  
    if(msgctl(msgid, IPC_RMID, 0) == -1)  
    {  
        fprintf(stderr, "msgctl(IPC_RMID) failedn");  
        exit(EXIT_FAILURE);  
    }  
    exit(EXIT_SUCCESS);  
}
```

发送信息的程序的源文件 msgsend.c 的源代码为：

```c
#include <unistd.h>  
#include <stdlib.h>  
#include <stdio.h>  
#include <string.h>  
#include <sys/msg.h>  
#include <errno.h>  
      
#define MAX_TEXT 512  
struct msg_st  
{  
    long int msg_type;  
    char text[MAX_TEXT];  
};  
      
int main()  
{  
    int running = 1;  
    struct msg_st data;  
    char buffer[BUFSIZ];  
    int msgid = -1;  
      
    //建立消息队列  
    msgid = msgget((key_t)1234, 0666 | IPC_CREAT);  
    if(msgid == -1)  
    {  
        fprintf(stderr, "msgget failed with error: %dn", errno);  
        exit(EXIT_FAILURE);  
    }  
      
    //向消息队列中写消息，直到写入end  
    while(running)  
    {  
        //输入数据  
        printf("Enter some text: ");  
        fgets(buffer, BUFSIZ, stdin);  
        data.msg_type = 1;    //注意2  
        strcpy(data.text, buffer);  
        //向队列发送数据  
        if(msgsnd(msgid, (void*)&data, MAX_TEXT, 0) == -1)  
        {  
            fprintf(stderr, "msgsnd failedn");  
            exit(EXIT_FAILURE);  
        }  
        //输入end结束输入  
        if(strncmp(buffer, "end", 3) == 0)  
            running = 0;  
        sleep(1);  
    }  
    exit(EXIT_SUCCESS);  
}

```

# 消息类型

这里主要说明一下消息类型是怎么一回事，注意 `msgreceive.`c 文件 `main` 函数中定义的变量 `msgtype`（注释为注意1），它作为 `msgrcv` 函数的接收信息类型参数的值，其值为0，表示获取队列中第一个可用的消息。再来看看 `msgsend.c` 文件中 `while` 循环中的语句 `data.msg_type = 1`（注释为注意2），它用来设置发送的信息的信息类型，即其发送的信息的类型为1。所以程序 `msgreceive` 能够接收到程序 `msgsend` 发送的信息。

如果把注意1，即 `msgreceive.c` 文件 main 函数中的语句由`long int msgtype = 0;` 改变为l `long int msgtype = 2;`会发生什么情况，msgreceive 将不能接收到程序msgsend发送的信息。因为在调用 `msgrcv` 函数时，如果 `msgtype`（第四个参数）大于零，则将只获取具有相同消息类型的第一个消息，修改后获取的消息类型为2，而 msg`send 发送的消息类型为1，所以不能被 `msgreceive` 程序接收。重新编译 `msgreceive.c` 文件并再次执行.

# 消息队列与命名管道的比较

消息队列跟命名管道有不少的相同之处，通过与命名管道一样，消息队列进行通信的进程可以是不相关的进程，同时它们都是通过发送和接收的方式来传递数据的。在命名管道中，发送数据用write，接收数据用read，则在消息队列中，发送数据用msgsnd，接收数据用msgrcv。而且它们对每个数据都有一个最大长度的限制。

与命名管道相比，消息队列的优势在于

1、消息队列也可以独立于发送和接收进程而存在，从而消除了在同步命名管道的打开和关闭时可能产生的困难。

2、同时通过发送消息还可以避免命名管道的同步和阻塞问题，不需要由进程自己来提供同步方法。

3、接收程序可以通过消息类型有选择地接收数据，而不是像命名管道中那样，只能默认地接收。

# 可能存在的隐患

当 64 位应用向 32 位应用发送一消息时，如果它在8字节字段中设置的值大于32位应用中4字节类型字段可表示值，那么32位应用在其 `mtype` 字段中得到的是一个截短了的值，于是也就丢失了信息。    











