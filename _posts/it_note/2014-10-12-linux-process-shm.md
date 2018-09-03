---
layout: post
title: linux进程间通信-共享内存
date: 2014-10-12 19:08:39
categories: Linux
tags: 系统编程
excerpt: linux process communication about share memory
---

### 一 共享内存介绍

共享内存可以从字面上去理解，就把一片逻辑内存共享出来，让不同的进程去访问它，修改它。共享内存是在两个正在运行的进程之间共享和传递数据的一种非常有效的方式。不同进程之间共享的内存通常安排为同一段物理内存。进程可以将同一段共享内存连接到它们自己的地址空间中，所有进程都可以访问共享内存中的地址，就好像它们是由用C语言函数malloc分配的内存一样。而如果某个进程向共享内存写入数据，所做的改动将立即影响到可以访问同一段共享内存的任何其他进程。
但有一点特别要注意：共享内存并未提供同步机制。也就是说，在第一个进程结束对共享内存的写操作之前，并无自动机制可以阻止第二个进程开始对它进行读取。所以我们通常需要用其他的机制来同步对共享内存的访问，例如信号量。

### 二 共享内存的使用
 
(1)创建共享内存

```c
 int shmget(key_t key, size_t size, int shmflg);
```

* key共享内存段的命名，shmget函数成功时返回一个与key相关的共享内存标识符（非负整数），用于后续的共享内存函数。调用失败返回-1.其它的进程可以通过该函数的返回值访问同一共享内存，它代表进程可能要使用的某个资源，程序对所有共享内存的访问都是间接的，程序先通过调用shmget函数并提供一个键，再由系统生成一个相应的共享内存标识符（shmget函数的返回值），只有shmget函数才直接使用信号量键，所有其他的信号量函数使用由semget函数返回的信号量标识符。
* size以字节为单位指定需要共享的内存容量。
* shmflg是权限标志，它的作用与open函数的mode参数一样，如果要想在key标识的共享内存不存在时，创建它的话，可以与IPC_CREAT做或操作。共享内存的权限标志与文件的读写权限一样，举例来说，0644,它表示允许一个进程创建的共享内存被内存创建者所拥有的进程向共享内存读取和写入数据，同时其他用户创建的进程只能读取共享内存。

(2)启动对该共享内存的访问

```c
void *shmat(int shm_id, const void *shm_addr, int shmflg);
```

第一次创建完共享内存时，它还不能被任何进程访问，shmat函数的作用就是用来启动对该共享内存的访问，并把共享内存连接到当前进程的地址空间。
* shm_id是由shmget函数返回的共享内存标识。
* shm_addr指定共享内存连接到当前进程中的地址位置，通常为空，表示让系统来选择共享内存的地址。
* shm_flg是一组标志位，通常为0。
调用成功时返回一个指向共享内存第一个字节的指针，如果调用失败返回-1.

(3)将共享内存从当前进程中分离

```c
int shmdt(const void *shmaddr);
```

该函数用于将共享内存从当前进程中分离。注意，将共享内存分离并不是删除它，只是使该共享内存对当前进程不再可用。
* shmaddr是shmat函数返回的地址指针，调用成功时返回0，失败时返回-1。
 
(4)控制共享内存

```c
int shmctl(int shm_id, int command, struct shmid_ds *buf);
```

shm_id是shmget函数返回的共享内存标识符。
command是要采取的操作，它可以取下面的三个值 ：
 *IPC_STAT：把shmid_ds结构中的数据设置为共享内存的当前关联值，即用共享内存的当前关联值覆盖shmid_ds的值。
 *IPC_SET：如果进程有足够的权限，就把共享内存的当前关联值设置为shmid_ds结构中给出的值
 *IPC_RMID：删除共享内存段
buf是一个结构指针，它指向共享内存模式和访问权限的结构。

```c
struct shmid_ds
{
     uid_t shm_perm.uid;
     uid_t shm_perm.gid;
     mode_t shm_perm.mode;
｝
```

### 三 例子

shmdata.h的源码:

```c
#ifndef _SHMDATA_H_HEADER
#define _SHMDATA_H_HEADER
#define TEXT_SZ 2048

struct shared_use_st
{
    int written;/* 作为一个标志，非0：表示可读，0表示可写 */
    char text[TEXT_SZ];/* 记录写入和读取的文本 */
};

#endif
```

 shmread.c的源代码

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/shm.h>
#include "shmdata.h"

#define MEM_KEY      (1234)

int main()
{
    int running = 1;             //程序是否继续运行的标志
    void *shm   = NULL;          //分配的共享内存的原始首地址
    struct shared_use_st *shared;//指向shm
    int shmid;                   //共享内存标识符
    //创建共享内存

    shmid = shmget((key_t)MEM_KEY, sizeof(struct shared_use_st), 0666|IPC_CREAT);
    if(shmid == -1)
    {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }
    //将共享内存连接到当前进程的地址空间
    shm = shmat(shmid, 0, 0);
    if(shm == (void*)-1)
    {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }
    printf("\nMemory attached at %X\n", (int)shm);
    //设置共享内存
    shared = (struct shared_use_st*)shm;
    shared->written = 0;
    while(running)//读取共享内存中的数据
    {
        //没有进程向共享内存定数据有数据可读取
        if(shared->written != 0)
        {
            printf("You wrote: %s", shared->text);
            sleep(rand() % 3);
            //读取完数据，设置written使共享内存段可写
            shared->written = 0;
            //输入了end，退出循环（程序）
            if(strncmp(shared->text, "end", 3) == 0)
                running = 0;
        }
        else//有其他进程在写数据，不能读取数据
            sleep(1);
    }
    //把共享内存从当前进程中分离
    if(shmdt(shm) == -1)
    {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }
    //删除共享内存
    if(shmctl(shmid, IPC_RMID, 0) == -1)
    {
        fprintf(stderr, "shmctl(IPC_RMID) failed\n");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}
```

shmwrite.c的源代码

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>
#include "shmdata.h"
 
#define MEM_KEY      (1234)

int main()
{
    int running = 1;
    void *shm = NULL;
    struct shared_use_st *shared = NULL;
    char buffer[BUFSIZ + 1];//用于保存输入的文本
    int shmid;
    //创建共享内存
    shmid = shmget((key_t)MEM_KEY, sizeof(struct shared_use_st), 0666|IPC_CREAT);
    if(shmid == -1)
    {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }
    //将共享内存连接到当前进程的地址空间
    shm = shmat(shmid, (void*)0, 0);
    if(shm == (void*)-1)
    {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }
    printf("Memory attached at %X\n", (int)shm);
    //设置共享内存
    shared = (struct shared_use_st*)shm;
    while(running)//向共享内存中写数据
    {
        //数据还没有被读取，则等待数据被读取,不能向共享内存中写入文本
        while(shared->written == 1)
        {
            sleep(1);
            printf("Waiting...\n");
        }
    //向共享内存中写入数据
    printf("Enter some text: ");
    fgets(buffer, BUFSIZ, stdin);
    strncpy(shared->text, buffer, TEXT_SZ);
    //写完数据，设置written使共享内存段可读
    shared->written = 1;
    //输入了end，退出循环（程序）
    if(strncmp(buffer, "end", 3) == 0)
    running = 0;
    }
    //把共享内存从当前进程中分离
    if(shmdt(shm) == -1)
    {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }
        sleep(2);
        exit(EXIT_SUCCESS);
}
```

**代码分析：**
(1)程序shmread创建共享内存，然后将它连接到自己的地址空间。在共享内存的开始处使用了一个结构struct_use_st。该结构中有个标志written，当共享内存中有其他进程向它写入数据时，共享内存中的written被设置为0，程序等待。当它不为0时，表示没有进程对共享内存写入数据，程序就从共享内存中读取数据并输出，然后重置设置共享内存中的written为0，即让其可被shmwrite进程写入数据。
(2)程序shmwrite取得共享内存并连接到自己的地址空间中。检查共享内存中的written，是否为0，若不是，表示共享内存中的数据还没有被完，则等待其他进程读取完成，并提示用户等待。若共享内存的written为0，表示没有其他进程对共享内存进行读取，则提示用户输入文本，并再次设置共享内存中的written为1，表示写完成，其他进程可对共享内存进行读操作。

**关于前面的例子的安全性讨论**
这个程序是不安全的，当有多个程序同时向共享内存中读写数据时，问题就会出现。可能你会认为，可以改变一下written的使用方式，例如，只有当written为0时进程才可以向共享内存写入数据，而当一个进程只有在written不为0时才能对其进行读取，同时把written进行加1操作，读取完后进行减1操作。这就有点像文件锁中的读写锁的功能。咋看之下，它似乎能行得通。但是这都不是原子操作，所以这种做法是行不能的。试想当written为0时，如果有两个进程同时访问共享内存，它们就会发现written为0，于是两个进程都对其进行写操作，显然不行。当written为1时，有两个进程同时对共享内存进行读操作时也是如些，当这两个进程都读取完是，written就变成了-1.
要想让程序安全地执行，就要有一种进程同步的进制，保证在进入临界区的操作是原子操作。例如，可以使用前面所讲的信号量来进行进程的同步。因为信号量的操作都是原子性的。

**使用共享内存的优缺点**
(1)优点：我们可以看到使用共享内存进行进程间的通信真的是非常方便，而且函数的接口也简单，数据的共享还使进程间的数据不用传送，而是直接访问内存，也加快了程序的效率。同时，它也不像匿名管道那样要求通信的进程有一定的父子关系。
(2)缺点：共享内存没有提供同步的机制，这使得我们在使用共享内存进行进程间通信时，往往要借助其他的手段来进行进程间的同步工作。

###四 更好的例子

1、server.c

```c
/*server.c:向共享内存中写入People*/
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <string.h>
#include "credis.h"
int semid;
int shmid;
/*信号量的P操作*/
void p()
{
    struct sembuf sem_p;
    sem_p.sem_num=0;/*设置哪个信号量*/
    sem_p.sem_op=-1;/*定义操作*/   
    if(semop(semid,&sem_p,1)==-1)
        printf("p operation is fail\n");  
    /*semop函数自动执行信号量集合上的操作数组。 
　　 int semop(int semid, struct sembuf semoparray[], size_t nops); 
　　 semoparray是一个指针，它指向一个信号量操作数组。nops规定该数组中操作的数量。*/ 
}
/*信号量的V操作*/
void v()
{
    struct sembuf sem_v;
    sem_v.sem_num=0;
    sem_v.sem_op=1;
    if(semop(semid,&sem_v,1)==-1)
        printf("v operation is fail\n");
}
int main()
{
    struct People{
        char name[10];
        int age;
    };
    key_t semkey;
    key_t shmkey;
    semkey=ftok("../test/VenusDB.cbp",0);       //用来产生唯一的标志符，便于区分信号量及共享内存
    shmkey=ftok("../test/main.c",0);
    /*创建信号量的XSI IPC*/
    semid=semget(semkey,1,0666|IPC_CREAT);//参数nsems,此时为中间值1，指定信号灯集包含信号灯的数目
    //0666|IPC_CREAT用来表示对信号灯的读写权限
    /*
    从左向右:
    第一位:0表示这是一个8进制数
    第二位:当前用户的经权限:6=110(二进制),每一位分别对就 可读,可写,可执行,6说明当前用户可读可写不可执行
    第三位:group组用户,6的意义同上
    第四位:其它用户,每一位的意义同上,0表示不可读不可写也不可执行
    */
    if(semid==-1)
        printf("creat sem is fail\n");
    //创建共享内存
    shmid=shmget(shmkey,1024,0666|IPC_CREAT);//对共享内存
    if(shmid==-1)
        printf("creat shm is fail\n");
    /*设置信号量的初始值，就是资源个数*/
    union semun{
        int val;
        struct semid_ds *buf;
        unsigned short *array;
    }sem_u;
    sem_u.val=1;    /*设置变量值*/
    semctl(semid,0,SETVAL,sem_u);   //初始化信号量，设置第0个信号量，p()操作为非阻塞的
    /*将共享内存映射到当前进程的地址中，之后直接对进程中的地址addr操作就是对共享内存操作*/
    struct People *addr;
    addr=(struct People*)shmat(shmid,0,0);  //将共享内存映射到调用此函数的内存段
    if(addr==(struct People*)-1)
        printf("shm shmat is fail\n");
    /*向共享内存写入数据*/
    p();
    strcpy((*addr).name,"xiaoming");
/*注意：①此处只能给指针指向的地址直接赋值，不能在定义一个  struct People people_1;addr=&people_1;因为addr在addr=(struct People*)shmat(shmid,0,0);时,已经由系统自动分配了一个地址，这个地址与共享内存相关联，所以不能改变这个指针的指向，否则他将不指向共享内存，无法完成通信了。
注意：②给字符数组赋值的方法。刚才太虎了。。*/
    (*addr).age=10;
    v();
    /*将共享内存与当前进程断开*/
    if(shmdt(addr)==-1)
        printf("shmdt is fail\n");  
}
```

2、clinet.c

```c
/*client.c:从共享内存中读出People*/
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
int semid;
int shmid;
/*信号量的P操作*/
void p()
{
    struct sembuf sem_p;
    sem_p.sem_num=0;
    sem_p.sem_op=-1;
    if(semop(semid,&sem_p,1)==-1)
        printf("p operation is fail\n");  
}
/*信号量的V操作*/
void v()
{
    struct sembuf sem_v;
    sem_v.sem_num=0;
    sem_v.sem_op=1;
    if(semop(semid,&sem_v,1)==-1)
    printf("v operation is fail\n");
}
int main()
{
    key_t semkey;
    key_t shmkey;
    semkey=ftok("../test/client/VenusDB.cbp",0);
    shmkey=ftok("../test/client/main.c",0);
    struct People{
        char name[10];
        int age;
    };
    /*读取共享内存和信号量的IPC*/ 
    semid=semget(semkey,0,0666);
    if(semid==-1)
        printf("creat sem is fail\n");
    shmid=shmget(shmkey,0,0666);
    if(shmid==-1)
        printf("creat shm is fail\n");
    /*将共享内存映射到当前进程的地址中，之后直接对进程中的地址addr操作就是对共享内存操作*/
    struct People *addr;
    addr=(struct People*)shmat(shmid,0,0);
    if(addr==(struct People*)-1)
        printf("shm shmat is fail\n");
    /*从共享内存读出数据*/
    p();
    printf("name:%s\n",addr->name);
    printf("age:%d\n",addr->age);
    v();
    /*将共享内存与当前进程断开*/
    if(shmdt(addr)==-1)
        printf("shmdt is fail\n");
 
    /*IPC必须显示删除。否则会一直留存在系统中*/
    if(semctl(semid,0,IPC_RMID,0)==-1)
        printf("semctl delete error\n");
    if(shmctl(shmid,IPC_RMID,NULL)==-1)
        printf("shmctl delete error\n");
}
```

### 五 父子进程共享内存的例子

```c
#include <stdio.h>  
#include <sys/types.h>  
#include <sys/ipc.h>  
#include <sys/sem.h>  
  
#define SHM_KEY 0x33  
#define SEM_KEY 0x44  
  
union semun {  
    int val;  
    struct semid_ds *buf;  
    unsigned short *array;  
};  
  
int P(int semid)  
{  
    struct sembuf sb;  
    sb.sem_num = 0;  
    sb.sem_op = -1;  
    sb.sem_flg = SEM_UNDO;  
      
    if(semop(semid, &sb, 1) == -1) {  
        perror("semop");  
        return -1;  
    }  
    return 0;  
}  
  
int V(int semid)  
{  
    struct sembuf sb;  
    sb.sem_num = 0;  
    sb.sem_op = 1;  
    sb.sem_flg = SEM_UNDO;  
      
    if(semop(semid, &sb, 1) == -1) {  
        perror("semop");  
        return -1;  
    }  
    return 0;  
}  
  
int main(int argc, char **argv)  
{  
    pid_t pid;  
    int i, shmid, semid;  
    int *ptr;  
    union semun semopts;  
  
    /* 创建一块共享内存, 存一个int变量 */  
    if ((shmid = shmget(SHM_KEY, sizeof(int), IPC_CREAT | 0600)) == -1) {  
        perror("msgget");  
    }  
    /* 将共享内存映射到进程, fork后子进程可以继承映射 */  
    if ((ptr = (int *)shmat(shmid, NULL, 0)) == (void *)-1) {  
        perror("shmat");  
    }  
    *ptr = 0;  
    /* 创建一个信号量用来同步共享内存的操作 */  
    if ((semid = semget(SEM_KEY, 1, IPC_CREAT | 0600)) == -1) {  
        perror("semget");  
    }  
    /* 初始化信号量 */  
    semopts.val = 1;  
    if (semctl(semid, 0, SETVAL, semopts) < 0) {  
        perror("semctl");  
    }  
    if ((pid = fork()) < 0) {  
        perror("fork");  
    } else if (pid == 0) {      /* Child */  
        /* 子进程对共享内存加1 */  
        for (i = 0; i < 100; i++) {  
            P(semid);  
            (*ptr)++;  
            V(semid);  
            printf("child: %d\n", *ptr);  
        }  
    } else {                    /* Parent */  
        /* 父进程对共享内存减1 */  
        for (i = 0; i < 100; i++) {  
            P(semid);  
            (*ptr)--;  
            V(semid);  
            printf("parent: %d\n", *ptr);  
        }  
        waitpid(pid);  
        sleep(2);
        /* 如果同步成功, 共享内存的值为0 */  
        printf("finally: %d\n", *ptr);  
    }  
  
    return 0;  
}  
```

