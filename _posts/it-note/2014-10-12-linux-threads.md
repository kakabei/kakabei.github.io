---
layout: post
title: linux 线程基础
date: 2014-10-12 21:28:17
categories: Linux
tags: 系统编程
excerpt: linux thread
---

# 一 程序的执行路线
 
包含#include<pthread.h>
创建线程的函数说明：

```c
int pthread_create(pthread_t *thread, pthread_addr_t *arr,void* (*start_routine)(void *), void *arg);
```
thread     	　：用于返回创建的线程的ID。
arr           : 用于指定的被创建的线程的属性，上面的函数中使用NULL，表示使用默认的属性。
start_routine   : 这是一个函数指针，指向线程被创建后要调用的函数。
arg　　　　　 : 用于给线程传递参数，在本例中没有传递参数，所以使用了NULL.

所有线程都有一个线程号，也就是Thread ID。其类型为pthread_t。通过调用pthread_self()函数可以获得自身的线程号。
pthread_join使一个线程等待另一个线程结束。代码中如果没有pthread_join主线程会很快结束从而使整个进程结束，从而使创建的线程没有机会开始执行就结束了。加入pthread_join后，主线程会一直等待直到等待的线程结束自己才结束，使创建的线程有机会执行。
函数定义：

```c
int pthread_join(pthread_t thread, void **retval);
```
thread: 线程标识符，即线程ID，标识唯一线程。
retval: 用户定义的指针，用来存储被等待线程的返回值。
返回值 ： 0代表成功。 失败，返回的则是错误号。
 
pthread_exit退出线程，线程的主动行为；由于一个进程中的多个线程是共享数据段的，因此通常在线程退出之后，退出线程所占用的资源并不会随着线程的终止而得到释放，但是可以用pthread_join()函数来同步并释放资源。

```c
void  pthread_exit（void  *retval）
```

retval：pthread_exit()调用线程的返回值，也可由函数pthread_join来获取.
 
互斥锁，多线程中保证共享数据操作的完整性。
线程私有：线程id，栈空间，执行序列，寄存器，调度优先级

例子：

```c
#include  <stdio.h>
#include  <pthread.h>
#include  <ctype.h>
 
int  total_words ;           /* the counter and its lock */
pthread_mutex_t counter_lock = PTHREAD_MUTEX_INITIALIZER;
main(int ac, char *av[])
{
    pthread_t  t1, t2;	 /* two threads */
    void *count_words(void *);
    if ( ac != 3 )
    {
     printf("usage: %s file1 file2\n", av[0]);
     exit(1);
    }
    total_words = 0;
    pthread_create(&t1, NULL, count_words, (void *) av[1]);
    pthread_create(&t2, NULL, count_words, (void *) av[2]);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    printf("%5d: total words\n", total_words);
}

void *count_words(void *f)
{
    char *filename = (char *) f;
    FILE *fp;
    int  c, prevc = '\0';
    if ((fp = fopen(filename, "r")) != NULL)
    {
        while( ( c = getc(fp)) != EOF )
        {
            if (!isalnum(c) && isalnum(prevc))
            {
                pthread_mutex_lock(&counter_lock);
                total_words++;
                pthread_mutex_unlock(&counter_lock);
            }
            prevc = c;
         }
         fclose(fp);
    }
     else 
   ｛
        perror(filename);
    ｝
    return NULL;
}
```

* 当pthread_mutex_lock()返回时，该互斥锁已被锁定。线程调用该函数让互斥锁上锁，如果该互斥锁已被另一个线程锁定和拥有，则调用该线程将阻塞，直到该互斥锁变为可用为止。 对于Solaris线程。
* 如果互斥锁类型为 PTHREAD_MUTEX_NORMAL，则不提供死锁检测。尝试重新锁定互斥锁会导致死锁。如果某个线程尝试解除锁定的互斥锁不是由该线程锁定或未锁定，则将产生不确定的行为。
* 如果互斥锁类型为 PTHREAD_MUTEX_ERRORCHECK，则会提供错误检查。如果某个线程尝试重新锁定的互斥锁已经由该线程锁定，则将返回错误。如果某个线程尝试解除锁定的互斥锁不是由该线程锁定或者未锁定，则将返回错误。
如果互斥锁类型为 PTHREAD_MUTEX_RECURSIVE，则该互斥锁会保留锁定计数这一概念。线程首次成功获取互斥锁时，锁定计数会设置为 1。线程每重新锁定该互斥锁一次，锁定计数就增加 1。线程每解除锁定该互斥锁一次，锁定计数就减小 1。 锁定计数达到 0 时，该互斥锁即可供其他线程获取。如果某个线程尝试解除锁定的互斥锁不是由该线程锁定或者未锁定，则将返回错误。
* 如果互斥锁类型是 PTHREAD_MUTEX_DEFAULT，则尝试以递归方式锁定该互斥锁将产生不确定的行为。对于不是由调用线程锁定的互斥锁，如果尝试解除对它的锁定，则会产生不确定的行为。如果尝试解除锁定尚未锁定的互斥锁，则会产生不确定的行为。

# 二 线程和进程

 进程与线程有根本的不同。每个进程有其独立的数据空间、文件描述符以及进程的ID。而线程共享一个数据空间，文件描述符以及进程ID。

（1） 一个进程至少包含一个线程。

（2） 线程有自己的栈，这个栈仍然是使用进程的地址空间，只是这块空间被线程标记为了栈。每个线程都会有自己私有的栈，这个栈是不可以被其他线程所访问的。

（3） 进程是程序执行时的一个实例。从内核的观点看，进程的目的就是担当分配系统资源（CPU时间、内存等）的基本单位。

（4） 线程是进程的一个执行流，是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位。

（5） 一个进程由几个线程组成（拥有很多相对独立的执行流的用户程序共享应用程序的大部分数据结构），线程与同属一个进程的其他的线程共享进程所拥有的全部资源。

（6） 进程：资源分配的最小单位，线程：程序执行的最小单位。

# 三 线程同步

1 条件变量

同步就是线程等待某个事件的发生。只有当等待的事件发生线程才继续执行，否则线程挂起并放弃处理器。当多个线程协作时，相互作用的任务必须在一定的条件下同步。
Linux下的C语言编程有多种线程同步机制，最典型的是条件变量(condition variable)。pthread_cond_init用来创建一个条件变量，其函数原型为：

```c
pthread_cond_init (pthread_cond_t *cond, const pthread_condattr_t *attr);
```

pthread_cond_wait和pthread_cond_timedwait用来等待条件变量被设置，值得注意的是这两个等待调用需要一个已经上锁的互斥体mutex，这是为了防止在真正进入等待状态之前别的线程有可能设置该条件变量而产生竞争。
pthread_cond_wait的函数原型为：

```c
pthread_cond_wait (pthread_cond_t *cond, pthread_mutex_t *mutex);
```

pthread_cond_broadcast用于设置条件变量，即使得事件发生，这样等待该事件的线程将不再阻塞。原型：

```c
pthread_cond_broadcast (pthread_cond_t *cond) ;
```

pthread_cond_signal则用于解除某一个等待线程的阻塞状态。原型：

```c
pthread_cond_signal (pthread_cond_t *cond) ;
```

pthread_cond_destroy 则用于释放一个条件变量的资源。

2、线程中的信号量

在头文件semaphore.h 中定义的信号量则完成了互斥体和条件变量的封装，按照多线程程序设计中访问控制机制，控制对资源的同步访问，提供程序设计人员更方便的调用接口。

```c
sem_init(sem_t *sem, int pshared, unsigned int val);
```

这个函数初始化一个信号量sem 的值为val，参数pshared 是共享属性控制，表明是否在进程间共享。

```c
sem_wait(sem_t *sem);
```

调用该函数时，若sem为无状态，调用线程阻塞，等待信号量sem值增加(post )成为有信号状态；若sem为有状态，调用线程顺序执行，但信号量的值减一。

```c
sem_post(sem_t *sem);
```

调用该函数，信号量sem的值增加，可以从无信号状态变为有信号状态。

# 四 例子

```c
#include <stdio.h>
#include <pthread.h>
#define BUFFER_SIZE 16 // 缓冲区数量

struct prodcons
{
    // 缓冲区相关数据结构
    int buffer[BUFFER_SIZE]; /* 实际数据存放的数组 */
    pthread_mutex_t lock; /* 互斥体lock 用于对缓冲区的互斥操作 */
    int readpos, writepos; /* 读写指针*/
    pthread_cond_t notempty; /* 缓冲区非空的条件变量 */
    pthread_cond_t notfull; /* 缓冲区未满的条件变量 */
};

/* 初始化缓冲区结构 */
void init(struct prodcons *b)
{
    pthread_mutex_init(&b->lock, NULL);
    pthread_cond_init(&b->notempty, NULL);
    pthread_cond_init(&b->notfull, NULL);
    b->readpos = 0;
    b->writepos = 0;
}

/* 将产品放入缓冲区,这里是存入一个整数 */
void put(struct prodcons *b, int data)
{
    pthread_mutex_lock(&b->lock);
    /* 等待缓冲区未满 */
    if ((b->writepos + 1) % BUFFER_SIZE == b->readpos)
    {
        pthread_cond_wait(&b->notfull, &b->lock);
    }
    /* 写数据,并移动指针 */
    b->buffer[b->writepos] = data;
    b->writepos++;
    if (b->writepos >= BUFFER_SIZE)
        b->writepos = 0;
    /* 设置缓冲区非空的条件变量*/
    pthread_cond_signal(&b->notempty);
    pthread_mutex_unlock(&b->lock);
} 

/* 从缓冲区中取出整数 */
int get(struct prodcons *b)
{
    int data;
    pthread_mutex_lock(&b->lock);
    /* 等待缓冲区非空*/
    if (b->writepos == b->readpos)
    {
        pthread_cond_wait(&b->notempty, &b->lock);
    }
    /* 读数据,移动读指针*/
    data = b->buffer[b->readpos];
    b->readpos++;
    if (b->readpos >= BUFFER_SIZE)
        b->readpos = 0;
    /* 设置缓冲区未满的条件变量*/
    pthread_cond_signal(&b->notfull);
    pthread_mutex_unlock(&b->lock);
    return data;
}

/* 测试:生产者线程将1 到10000 的整数送入缓冲区,消费者线
   程从缓冲区中获取整数,两者都打印信息*/
#define OVER ( - 1)
struct prodcons buffer;
void *producer(void *data)
{
    int n;
    for (n = 0; n < 10000; n++)
    {
        printf("%d --->\n", n);
        put(&buffer, n);
    } put(&buffer, OVER);
    return NULL;
}
void *consumer(void *data)
{
    int d;
    while (1)
    {
        d = get(&buffer);
        if (d == OVER)
            break;
        printf("--->%d \n", d);
    }
    return NULL;
}

int main(void)
{
    pthread_t th_a, th_b;
    void *retval;
    init(&buffer);
    /* 创建生产者和消费者线程*/
    pthread_create(&th_a, NULL, producer, 0);
    pthread_create(&th_b, NULL, consumer, 0);
    /* 等待两个线程结束*/
    pthread_join(th_a, &retval);
    pthread_join(th_b, &retval);
    return 0;
}
```

# 五 使用线程的好处

（1）和进程相比，它是一种非常"节俭"的多任务操作方式。在Linux系统下，启动一个新的进程必须分配给它独立的地址空间，建立众多的数据表来维护它的代码段、堆栈段和数据段，这是一种"昂贵"的多任务工作方式。而运行于一个进程中的多个线程，它们彼此之间使用相同的地址空间，共享大部分数据，启动一个线程所花费的空间远远小于启动一个进程所花费的空间，而且，线程间彼此切换所需的时间也远远小于进程间切换所需要的时间。一个进程的开销大约是一个线程开销的30倍左右。
（2）线程间方便的通信机制。对不同进程来说，它们具有独立的数据空间，要进行数据的传递只能通过通信的方式进行，这种方式不仅费时，而且很不方便。线程则不然，由于同一进程下的线程之间共享数据空间，所以一个线程的数据可以直接为其它线程所用，这不仅快捷，而且方便。当然，数据的共享也带来其他一些问题，有的变量不能同时被两个线程所修改，有的子程序中声明为static的数据更有可能给多线程程序带来灾难性的打击，这些正是编写多线程程序时最需要注意的地方。
（3）提高应用程序响应。这对图形界面的程序尤其有意义，当一个操作耗时很长时，整个系统都会等待这个操作，此时程序不会响应键盘、鼠标、菜单的操作，而使用多线程技术，将耗时长的操作（time consuming）置于一个新的线程，可以避免这种尴尬的情况。
（4）使多CPU系统更加有效。操作系统会保证当线程数不大于CPU数目时，不同的线程运行于不同的CPU上。
（5）改善程序结构。一个既长又复杂的进程可以考虑分为多个线程，成为几个独立或半独立的运行部分，这样的程序会利于理解和修改。

# 六 linux gcc编译问题
在用gcc编译和线程相关的程序时，报了如下的错误：

```sh
[root@192 process]# gcc pthread.c -o pthread
/tmp/ccJKG8rq.o: In function `main':
pthread.c:(.text+0x29c): undefined reference to `pthread_create'
pthread.c:(.text+0x2b7): undefined reference to `pthread_create'
pthread.c:(.text+0x2ca): undefined reference to `pthread_join'
pthread.c:(.text+0x2dd): undefined reference to `pthread_join'
collect2: ld returned 1 exit status
```
解决方案： 程序要引用线程相关的库。所以编译时加上-lpthread。即：gcc pthread.c -o pthread -lpthread
