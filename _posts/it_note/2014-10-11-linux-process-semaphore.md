---
layout: post
title: linux进程间通信-信号量（semaphore）
date: 2014-10-11 22:25:30
categories: Linux
tags: 系统编程
excerpt: linux process communication about  semaphore
---

### 一 为什么要使用信号量

为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行线程访问 代码的临界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个线程在访问它， 也就是说信号量是用来调协进程对共享资源的访问的。其中共享内存的使用就要用到信号量。

### 二 信号量的工作原理

由于信号量只能进行两种操作等待和发送信号，即P(sv)和V(sv),他们的行为是这样的：
P(sv)：如果sv的值大于零，就给它减1；如果它的值为零，就挂起该进程的执行
V(sv)：如果有其他进程因等待sv而被挂起，就让它恢复运行，如果没有进程因等待sv而挂起，就给它加1.

举个例子，就是 两个进程共享信号量sv，一旦其中一个进程执行了P(sv)操作，它将得到信号量，并可以进入临界区，使sv减1。而第二个进程将被阻止进入临界区，因为 当它试图执行P(sv)时，sv为0，它会被挂起以等待第一个进程离开临界区域并执行V(sv)释放信号量，这时第二个进程就可以恢复执行。

### 三 Linux的信号量机制

Linux提供了一组精心设计的信号量接口来对信号进行操作，它们不只是针对二进制信号量，下面将会对这些函数进行介绍，但请注意，这些函数都是用来对成组的信号量值进行操作的。它们声明在头文件sys/sem.h中。

### 四 信号号相关的两个结构体

内核为每个信号量集合设置了一个semid_ds结构

```c
struct semid_ds {
    struct ipc_permsem_perm ;
    structsem*    sem_base ; //信号数组指针
    ushort        sem_nsem ; //此集中信号个数
    time_t        sem_otime ; //最后一次semop时间
    time_t        sem_ctime ; //最后一次创建时间
} ;
```

每个信号量由一个无名结构表示，它至少包含下列成员： （这个是什么意思？？）

```c
struct  {
    ushort_t  semval ;  //信号量的值
    short     sempid ;  //最后一个调用semop的进程ID
    ushort    semncnt ; //等待该信号量值大于当前值的进程数（一有进程释放资源 就被唤醒）
    ushort    semzcnt ; //等待该信号量值等于0的进程数
} ; 
```

### 四 信号量的使用

1、创建信号量
semget函数创建一个信号量集或访问一个已存在的信号量集。

```c
#include <sys/sem.h>
int  semget (key_t key,  int nsem, int oflag) ;
```

返回值是一个称为信号量标识符的整数，semop和semctl函数将使用它。
参数nsem指定集合中的信号量数。（若用于访问一个已存在的集合，那就可以把该参数指定为0）
参数oflag可以是SEM_R(read)和SEM_A(alter)常值的组合。（打开时用到），也可以是IPC_CREAT或IPC_EXCL ;

2、打开信号量
使用semget打开一个信号量集后，对其中一个或多个信号量的操作就使用semop(op--operate)函数来执行。

```c
#include <sys/sem.h>
int  semop (int semid,  struct sembuf * opsptr,  size_t nops) ;
```

参数opsptr是一个指针，它指向一个信号量操作数组，信号量操作由sembuf结构表示：

```c
struct sembuf{  
    short sem_num;   // 除非使用一组信号量，否则它为0  
    short sem_op;    // 信号量在一次操作中需要改变的数据，通常是两个数，
                     // 一个是-1，即P（等待）操作，一个是+1，即V（发送信号）操作  
    short sem_flg;   // 通常为SEM_UNDO,使操作系统跟踪信号，并在进程没有释放该信号量而终止时，
                     // 操作系统释放信号量  
};  
```

* 参数nops规定opsptr数组中元素个数。
sem_op值：
（1）若sem_op为正，这对应于进程释放占用的资源数。sem_op值加到信号量的值上。（V操作）
（2）若sem_op为负,这表示要获取该信号量控制的资源数。信号量值减去sem_op的绝对值。（P操作）
（3）若sem_op为0,这表示调用进程希望等待到该信号量值变成0
* 如果信号量值小于sem_op的绝对值（资源不能满足要求），则：
（1）若指定了IPC_NOWAIT，则semop()出错返回EAGAIN。
（2）若未指定IPC_NOWAIT，则信号量的semncnt值加1（因为调用进程将进入休眠状态），然后调用进程被挂起直至：①此信号量变成大于或等于sem_op的绝对值；②从系统中删除了此信号量，返回EIDRM；③进程捕捉到一个信 号，并从信号处理程序返回，返回EINTR。（与消息队列的阻塞处理方式 很相似）

 3、信号量是操作
semctl函数对一个信号量执行各种控制操作。

```c
#include <sys/sem.h>
int  semctl (int semid,  int semnum, int cmd, /*可选参数*/ ) ;
```

第四个参数是可选的，取决于第三个参数cmd。
参数semnum指定信号集中的哪个信号（操作对象）
参数cmd指定以下10种命令中的一种,在semid指定的信号量集合上执行此命令。
IPC_STAT   读取一个信号量集的数据结构semid_ds，并将其存储在semun中的buf参数中。
IPC_SET     设置信号量集的数据结构semid_ds中的元素ipc_perm，其值取自semun中的buf参数。
IPC_RMID  将信号量集从内存中删除。
GETALL      用于读取信号量集中的所有信号量的值。
GETNCNT  返回正在等待资源的进程数目。
GETPID      返回最后一个执行semop操作的进程的PID。
GETVAL      返回信号量集中的一个单个的信号量的值。
GETZCNT   返回这在等待完全空闲的资源的进程数目。
SETALL       设置信号量集中的所有的信号量的值。
SETVAL      设置信号量集中的一个单独的信号量的值。

### 五 信号量值的初始化

semget并不初始化各个信号量的值，这个初始化必须通过以SETVAL命令(设置集合中的一个值)或SETALL命令(设置集合中的所有值) 调用semctl来完成。
SystemV信号量的设计中，创建一个信号量集并将它初始化需两次函数调用是一个致命的缺陷。一个不完备的解决方案是：在调用semget时指定IPC_CREAT | IPC_EXCL标志，这样只有一个进程（首先调用semget的那个进程）创建所需信号量，该进程随后初始化该信号量。

### 六 例子

```c
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/sem.h>

union semun
{
    int val;
    struct semid_ds *buf;
    unsigned short *arry;
};

static int sem_id = 0;
static int set_semvalue();
static void del_semvalue();
static int semaphore_p();
static int semaphore_v();

int main(int argc, char *argv[])
{
    char message = 'X';
    int i = 0;
    /* 创建信号量 */
    sem_id = semget((key_t)1234, 1, 0666 | IPC_CREAT);
    if(argc > 1)
    {
        /* 程序第一次被调用，初始化信号量 */
        if(!set_semvalue())
        {
            fprintf(stderr, "Failed to initialize semaphore\n");
            exit(EXIT_FAILURE);
        }
        /* 设置要输出到屏幕中的信息，即其参数的第一个字符 */
        message = argv[1][0];
        sleep(2);
    }
    for(i = 0; i < 10; ++i)
    {
        /* 进入临界区 */
        if(!semaphore_p())
        {
            exit(EXIT_FAILURE);
        }
        /* 向屏幕中输出数据 */
        printf("%c", message);
        /* 清理缓冲区，然后休眠随机时间 */
        fflush(stdout);
        sleep(rand() % 3);
        /* 离开临界区前再一次向屏幕输出数据 */
        printf("%c", message);
        fflush(stdout);
        /* 离开临界区，休眠随机时间后继续循环 */
        if(!semaphore_v())
        {
            exit(EXIT_FAILURE);
        }
        sleep(rand() % 2);
    }
    sleep(10);
    printf("\n%d - finished\n", getpid());
    if(argc > 1)
    {
        /* 如果程序是第一次被调用，则在退出前删除信号量 */
        sleep(3);
        del_semvalue();
    }
    exit(EXIT_SUCCESS);
}

static int set_semvalue()
{
    /* 用于初始化信号量，在使用信号量前必须这样做 */
    union semun sem_union;
    sem_union.val = 1;
    if(semctl(sem_id, 0, SETVAL, sem_union) == -1)
    {
        return 0;
    }
    return 1;
}

static void del_semvalue()
{
    /* 删除信号量 */
    union semun sem_union;
    if(semctl(sem_id, 0, IPC_RMID, sem_union) == -1)
    {
        fprintf(stderr, "Failed to delete semaphore\n");
    }
}

static int semaphore_p()
{
    /* 对信号量做减1操作，即等待P（sv）*/
    struct sembuf sem_b;
    sem_b.sem_num = 0;
    sem_b.sem_op = -1;//P()
    sem_b.sem_flg = SEM_UNDO;
    if(semop(sem_id, &sem_b, 1) == -1)
    {
        fprintf(stderr, "semaphore_p failed\n");
        return 0;
    }
    return 1;
}

static int semaphore_v()
{
    /* 这是一个释放操作，它使信号量变为可用，即发送信号V（sv）*/
    struct sembuf sem_b;
    sem_b.sem_num = 0;
    sem_b.sem_op = 1;//V()
    sem_b.sem_flg = SEM_UNDO;
    if(semop(sem_id, &sem_b, 1) == -1)
    {
        fprintf(stderr, "semaphore_v failed\n");
        return 0;
    }
    return 1;
}
```

### 七 信号量集合的例子

```c
#include<stdio.h>  
#include<sys/types.h>  
#include<sys/ipc.h>  
#include<sys/sem.h>  
#include<errno.h>  
#include<string.h>  
#include<stdlib.h>  
#include<assert.h>  
#include<time.h>  
#include<unistd.h>  
#include<sys/wait.h>    
#define MAX_SEMAPHORE 10   
#define FILE_NAME "test2.c"  
  
union semun{  
    int val ;  
    struct semid_ds *buf ;  
    unsigned short  *array ;  
    struct seminfo *_buf ;  
}arg;  
struct semid_ds sembuf;  
  
int main()  
{  
    key_t key ;  
    int semid ,ret,i;   
    unsigned short buf[MAX_SEMAPHORE] ;  
    struct sembuf sb[MAX_SEMAPHORE] ;  
    pid_t pid ;  
      
    pid = fork() ;  
    if(pid < 0)  
    {  
        /* Create process Error! */  
        fprintf(stderr,"Create Process Error!:%s\n",strerror(errno));  
        exit(1) ;  
    }     
    if(pid > 0)  
    {  
        /* in parent process !*/          
        key = ftok(FILE_NAME,'a') ;  
        if(key == -1)  
        {  
            /* in parent process*/  
            fprintf(stderr,"Error in ftok:%s!\n",strerror(errno));  
            exit(1) ;     
        }  
  
        semid = semget(key,MAX_SEMAPHORE,IPC_CREAT|0666);  //创建信号量集合
        if(semid == -1)  
        {  
            fprintf(stderr,"Error in semget:%s\n",strerror(errno));   
            exit(1) ;     
        }  
        printf("Semaphore have been initialed successfully in parent process,ID is :%d\n",semid);     
        sleep(2) ;  
        printf("parent wake up....\n");  
        /*父进程在子进程得到semaphore的时候请求semaphore，此时父进程将阻塞直至子进程释放掉semaphore*/  
        /* 此时父进程的阻塞是因为semaphore 1 不能申请，因而导致的进程阻塞*/  
        for(i=0;i<MAX_SEMAPHORE;++i)  
        {  
            sb[i].sem_num = i ;  
            sb[i].sem_op = -1 ;     /*表示申请semaphore*/  
            sb[i].sem_flg = 0 ;  
        }  
        printf("parent is asking for resource...\n");  
        ret = semop(semid , sb ,10); //p() 
        if(ret == 0)  
        {  
            printf("parent got the resource!\n");  
        }         
        /* 父进程等待子进程退出 */  
        waitpid(pid,NULL,0);  
        printf("parent exiting .. \n");  
        exit(0) ;  
    }  
    else  
    {  
        /* in child process! */   
        key = ftok(FILE_NAME,'a') ;  
        if(key == -1)  
        {  
            /* in child process*/  
            fprintf(stderr,"Error in ftok:%s!\n",strerror(errno));  
            exit(1) ;     
        }  
  
        semid = semget(key,MAX_SEMAPHORE,IPC_CREAT|0666);  
        if(semid == -1)  
        {  
            fprintf(stderr,"Error in semget:%s\n",strerror(errno));   
            exit(1) ;     
        }  
        printf("Semaphore have been initialed successfully in child process,ID is:%d\n",semid);   
  
        for(i=0;i<MAX_SEMAPHORE;++i)  
        {  
            /* Initial semaphore */  
            buf[i] = i + 1;  
        }  
        arg.array = buf;  
        ret = semctl(semid , 0, SETALL,arg);  
        if(ret == -1)  
        {  
            fprintf(stderr,"Error in semctl in child:%s!\n",strerror(errno));  
            exit(1) ;  
        }         
        printf("In child , Semaphore Initailed!\n");  
  
        /*子进程在初始化了semaphore之后，就申请获得semaphore*/  
        for(i=0;i<MAX_SEMAPHORE;++i)  
        {
            sb[i].sem_num = i ;  
            sb[i].sem_op = -1 ;  
            sb[i].sem_flg = 0 ;  
        }
        ret = semop(semid , sb , 10);//信号量0被阻塞
        if( ret == -1 )  
        {  
            fprintf(stderr,"子进程申请semaphore失败：%s\n",strerror(errno));  
            exit(1) ;         
        }         
        printf("child got semaphore,and start to sleep 3 seconds!\n");  
        sleep(3) ;    
        printf("child wake up .\n");  
        for(i=0;i < MAX_SEMAPHORE;++i)  
        {  
            sb[i].sem_num = i ;  
            sb[i].sem_op =  +1 ;  
            sb[i].sem_flg = 0 ;       
        }  
        printf("child start to release the resource...\n");  
        ret = semop(semid, sb ,10) ;      
        if(ret == -1)  
        {  
            fprintf(stderr,"子进程释放semaphore失败:%s\n",strerror(errno));  
            exit(1) ;  
        }         
        ret = semctl(semid ,0 ,IPC_RMID);  
        if(ret == -1)  
        {  
            fprintf(stderr,"semaphore删除失败:%s！\n",strerror(errno));  
            exit(1) ;     
        }         
        printf("child exiting successfully!\n");      
        exit(0) ;         
    }  
    return 0;  
}  
```

信号量的意图在于进程间同步，互斥锁和条件变量的意图则在于线程间同步。但是信号量也可用于线程间，互斥锁和条件变量也可用于进程间。我们应该使用适合具体应用的那组原语。
 

