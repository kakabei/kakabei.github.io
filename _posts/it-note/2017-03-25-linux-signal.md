---
layout: post
title: linux signal and threads
date: 2017-03-15 21:12:15
categories: Linux
tags: 系统编程 thread
excerpt: linux signal 笔记，主要是信号和多线程的一些知识点。
---

#### 信号与进程：

信号是向进程异步发送的软件通知，通知进程有事件发生。事件可为硬件异常(如除0)、软件条件(如闹钟超时)、控制终端发出的信号或调用kill()/raise()函数产生的用户逻辑信号。

1.当信号产生时，内核通常在进程表中设置一个某种形式的标志，即向进程递送一个信号。在信号产生(generation)和递送(delivery)之间(可能相当长)的时间间隔内，该信号处于未决(pending)状态。已经生成但未递送的信号称为挂起(suspending)的信号。[这里涉及了两个状态， 一个**未决**， 一个**挂起**]

2.进程可选择阻塞(block)某个信号，此时若对该信号的动作是系统默认动作或捕捉该信号，则为该进程将此信号保持为未决状态，直到该进程(a)对此信号解除阻塞，或者(b)将对此信号的动作更改为忽略。

3.内核为每个进程维护一个未决(未处理的)信号队列，信号产生时无论是否被阻塞，首先放入未决队列里。当时间片调度到当前进程时，内核检查未决队列中是否存在信号。
若有信号且未被阻塞，则执行相应的操作并从队列中删除该信号；否则仍保留该信号。 因此，进程在信号递送给它之前仍可改变对该信号的动作。进程调用sigpending()函数判定哪些信号设置为阻塞并处于未决状态。 **[产生信号后还没投递就会被放在未决信号队列中]**

4.若在进程解除对某信号的阻塞之前，该信号发生多次，则未决队列仅保留相同不可靠信号中的一个，而可靠信号(实时扩展)会保留并递送多次，称为按顺序排队。**[这个把信号分为可靠可靠的两种，那么哪一些是可靠的，那一些是不可靠呢？]**

5.每个进程都有一个信号屏蔽字(signal mask)，规定当前要阻塞递送到该进程的信号集。对于每个可能的信号，该屏蔽字中都有一位与之对应。对于某种信号，若其对应位已设置，则该信号当前被阻塞。**[在一个进程中对一个信号的阻塞就是通过设置这个进程的屏蔽字来完成的]**

#### 信号与线程

内核也为每个线程维护未决信号队列。当调用sigpending()时，返回**整个进程未决信号队列与调用线程未决信号队列的并集**。进程内创建线程时，新线程将继承进程(主线程)的信号屏蔽字，但新线程的未决信号集被清空(以防同一信号被多个线程处理)。**[新线程的未决信号集,其实就是新的未决信号队列]**
 
线程的信号屏蔽字是私有的(定义当前线程要求阻塞的信号集)，即线程可独立地屏蔽某些信号。这样，应用程序可控制哪些线程响应哪些信号。

信号处理函数由进程内所有线程共享。这意味着尽管单个线程可阻止某些信号，但当线程修改某信号相关的处理行为后，所有线程都共享该处理行为的改变。这样，若某线程选择忽略某信号，而其他线程可恢复信号的默认处理行为或为信号设置新的处理函数，从而撤销原先的忽略行为。即对某个信号处理函数，以最后一次注册的处理函数为准，从而保证同一信号被任意线程处理时行为相同。此外，若某信号的默认动作是停止或终止，则不管该信号发往哪个线程，整个进程都会停止或终止。

若信号与硬件故障(如SIGBUS/SIGFPE/SIGILL/SIGSEGV)或定时器超时相关，该信号会发往引起该事件的线程。其它信号除非显式指定目标线程，否则通常发往主线程(哪怕信号处理函数由其他线程注册)，仅当主线程屏蔽该信号时才发往某个具有处理能力的线程。

#### 接口

##### 一  pthread_sigmask

线程可调用pthread_sigmask()设置本线程的信号屏蔽字，以屏蔽该线程对某些信号的响应处理。

```c
#include <signal.h>
int pthread_sigmask(int how, const sigset_t *restrict set, sigset_t *restrict oset);
```
该函数检查和(或)更改本线程的信号屏蔽字。若参数oset为非空指针，则该指针返回调用前本线程的信号屏蔽字。若参数set为非空指针，则参数how指示如何修改当前信号屏蔽字；否则不改变本线程信号屏蔽字，并忽略how值。该函数执行成功时返回0，否则返回错误编号(errno)。

下列给出参数how可选用的值。其中，SIG_ BLOCK为“或”操作，而SIG_SETMASK为赋值操作。

- SIG_BLOCK 将set中包含的信号加入本线程的当前信号屏蔽字
- SIG_UNBLOCK 从本线程的当前信号屏蔽字中移除set中包含的信号(哪怕该信号并未被阻塞)
- SIG_SETMASK 将set指向的信号集设置为本线程的信号屏蔽字

主线程调用pthread_sigmask()设置信号屏蔽字后，其创建的新线程将继承主线程的信号屏蔽字。然而，新线程对信号屏蔽字的更改不会影响创建者和其他线程。

注意，pthread_sigmask()与sigprocmask()函数功能类似。两者的区别在于，pthread_sigmask()是线程库函数，用于多线程进程，且失败时返回errno；而sigprocmask()针对单线程的进程，其行为在多线程的进程中没有定义，且失败时设置errno并返回-1。

###### 二  sigwait

线程可通过调用sigwait()函数等待一个或多个信号发生。

```c
#include <signal.h>
int sigwait(const sigset_t *restrict sigset, int *restrict signop);
```

参数sigset指定线程等待的信号集，signop指向的整数表明接收到的信号值。

该函数将调用线程挂起，直到信号集中的任何一个信号被递送。该函数接收递送的信号后，将其从未决队列中移除(以防返回时信号被signal/sigaction安装的处理函数捕获)，然后唤醒线程并返回。

该函数执行成功时返回0，并将接收到的信号值存入signop所指向的内存空间；失败时返回错误编号(errno)。失败原因通常为EINVAL(指定信号无效或不支持)，但并不返回EINTR错误。
     
给定线程的未决信号集是整个进程未决信号集与该线程未决信号集的并集。若等待信号集中某个信号在sigwait()调用时处于未决状态，则该函数将无阻塞地返回。若同时有多个等待中的信号处于未决状态，则对这些信号的选择规则和顺序未定义。在返回之前，sigwait()将从进程中原子性地移除所选定的未决信号。
     
若已阻塞等待信号集中的信号，则sigwait()会自动解除信号集的阻塞状态，直到有新的信号被递送。在返回之前，sigwait()将恢复线程的信号屏蔽字。因此，sigwait()并不改变信号的阻塞状态。可见，sigwait()的这种**"解阻-等待-阻塞"**特性，与条件变量非常相似。**[为什么？后面例子说明的第9点有说到]**
     
为避免错误发生，调用sigwait()前必须阻塞那些它正在等待的信号。在单线程环境中，调用程序首先调用`sigprocmask()`阻塞等待信号集中的信号，以防这些信号在连续的sigwait()调用之间进入未决状态，从而触发默认动作或信号处理函数。在多线程程序中，所有线程(包括调用线程)都必须阻塞等待信号集中的信号，否则信号可能被递送到调用线程之外的其他线程。建议在创建线程前调用pthread_sigmask()阻塞这些信号(新线程继承信号屏蔽字)，然后绝不显式解除阻塞(sigwait会自动解除信号集的阻塞状态)。


若多个线程调用sigwait()等待同一信号，只有一个(但不确定哪个)线程可从sigwait()中返回。若信号被捕获(通过sigaction安装信号处理函数)，且线程正在sigwait()调用中等待同一信号，则由系统实现来决定以何种方式递送信号。**操作系统实现可让sigwait返回(通常优先级较高)**，也可激活信号处理程序，但不可能出现两者皆可的情况。

注意，sigwait()与sigwaitinfo()函数功能类似。两者的区别在于，sigwait()成功时返回0并传回信号值，且失败时返回errno；而sigwaitinfo()成功时返回信号值并传回siginfo_t结构(信息更多)，且失败时设置errno并返回-1。此外， 当产生等待信号集以外的信号时，该信号的处理函数可中断sigwaitinfo()，此时errno被设置为EINTR。对SIGKILL (杀死进程)和 SIGSTOP(暂停进程)信号的等待将被系统忽略。
     
使用sigwait()可简化多线程环境中的信号处理，允许在指定线程中以同步方式等待并处理异步产生的信号。为了防止信号中断线程，可将信号加到每个线程的信号屏蔽字中，然后安排专用线程作信号处理。该专用线程可进行任何函数调用，而不必考虑函数的可重入性和异步信号安全性，因为这些函数调用来自正常的线程环境，能够知道在何处被中断并继续执行。这样，信号到来时就不会打断其他线程的工作。

#### 信号处理模型

这种采用专用线程同步处理信号的模型如下图所示：

![](/assets/linux/linux_signal.jpeg) 

其设计步骤如下：
1. 主线程设置信号屏蔽字，阻塞希望同步处理的信号；
2. 主线程创建一个信号处理线程，该线程将希望同步处理的信号集作为 sigwait()的参数；
3. 主线程创建若干工作线程。
     
主线程的信号屏蔽字会被其创建的新线程继承，故工作线程将不会收到信号。

注意，因程序逻辑需要而产生的信号(如SIGUSR1/ SIGUSR2和实时信号)，被处理后程序继续正常运行，可考虑使用sigwait同步模型规避信号处理函数执行上下文不确定性带来的潜在风险。而对于硬件致命错误等导致程序运行终止的信号(如SIGSEGV)，必须按照传统的异步方式使用 signal()或sigaction()注册信号处理函数进行非阻塞处理，以提高响应的实时性。在应用程序中，可根据所处理信号的不同而同时使用这两种信号处理模型。

因为sigwait()以阻塞方式同步处理信号，为避免信号处理滞后或非实时信号丢失的情况，处理每个信号的代码应尽量简洁快速，避免调用会产生阻塞的库函数。

#### 例子

首先定义两个信号处理函数：

编译链接(加-pthread选项)后,  g++ -pthread signal.cpp -o signal


```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <pthread.h>
#include <signal.h>
#include <string.h>

static void SigHandler(int dwSigNo)
{
    printf("++Thread %x Received Signal %2d(%s)!\n",
          (unsigned int)pthread_self(), dwSigNo, strsignal(dwSigNo));
}
static void sighandler(int dwSigNo)
{   //非异步信号安全，仅为示例
    printf("--Thread %x Received Signal %2d(%s)!\n",
            (unsigned int)pthread_self(), dwSigNo, strsignal(dwSigNo));
}

int main(void)
 {
    sigset_t tBlockSigs;
    sigemptyset(&tBlockSigs);
    sigaddset(&tBlockSigs, SIGINT);
    sigprocmask(SIG_BLOCK, &tBlockSigs, NULL);

    signal(SIGQUIT, sighandler);

    int dwRet;
 #ifdef USE_SIGWAIT
    int dwSigNo;
    dwRet = sigwait(&tBlockSigs, &dwSigNo);
    printf("sigwait returns %d(%s), signo = %d\n", dwRet,  dwSigNo);
#else
    siginfo_t tSigInfo;
    dwRet = sigwaitinfo(&tBlockSigs, &tSigInfo);
    printf("sigwaitinfo returns %d(%s), signo = %d\n", dwRet,  tSigInfo.si_signo);
#endif
    return 0;

```
执行结果如下：

```sh 
// define USE_SIGWAIT
[root@VM_116_132_centos ~]# ./signal
^\--Thread 447c5740 Received Signal  3(Quit)!   // Ctrl + \
^\--Thread 447c5740 Received Signal  3(Quit)!   // Ctrl + \
^CSegmentation fault                            // Ctrl + C
[root@VM_116_132_centos ~]#

// no define USE_SIGWAIT
^\--Thread 3aa51740 Received Signal  3(Quit)! // Ctrl + \ 直接退出了
sigwaitinfo returns -1((null)), signo = -1

```

对比可见，sigwaitinfo()可被等待信号集以外的信号中断，而sigwait()不会被中断。**[为什么?]**

测试多线程中，sigwait()和sigwaitinfo()函数对信号的同步等待

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <pthread.h>
#include <signal.h>
#include <string.h>
#include <errno.h>
extern int errno;

//#define USE_SIGWAIT
static void SigHandler(int dwSigNo)
{
    printf("++Thread %x Received Signal %2d(%s)!\n",
          (unsigned int)pthread_self(), dwSigNo, strsignal(dwSigNo));
}
static void sighandler(int dwSigNo)
{   //非异步信号安全，仅为示例
    printf("--Thread %x Received Signal %2d(%s)!\n",
            (unsigned int)pthread_self(), dwSigNo, strsignal(dwSigNo));
}

void *SigMgrThread(void *pvArg)
{
    pthread_detach(pthread_self());

    //捕获SIGQUIT信号，以免程序收到该信号后退出
    signal(SIGQUIT, sighandler);
 
    //使用创建线程时的pvArg传递信号屏蔽字
    int dwRet;
    while(1)
    {
#ifdef USE_SIGWAIT
        int dwSigNo;
        dwRet = sigwait((sigset_t*)pvArg, &dwSigNo);
        if(dwRet == 0)
            SigHandler(dwSigNo);
        else
            printf("sigwait() failed, errno: %d(%s)!\n", dwRet, strerror(dwRet));
#else
        siginfo_t tSigInfo;
        dwRet = sigwaitinfo((sigset_t*)pvArg, &tSigInfo);
        if(dwRet != -1) //dwRet与tSigInfo.si_signo值相同
            SigHandler(tSigInfo.si_signo);
        else
        {
            if(errno == EINTR) //被其他信号中断
                printf("sigwaitinfo() was interrupted by a signal handler!\n");
            else
                printf("sigwaitinfo() failed, errno: %d(%s)!\n", errno, strerror(errno));
        }
    }
#endif
}

int ThreadKill(pthread_t tThrdId, int dwSigNo)
{
    int dwRet = pthread_kill(tThrdId, dwSigNo);
    if(dwRet == ESRCH)
        printf("Thread %x is non-existent(Never Created or Already Quit)!\n",
            (unsigned int)tThrdId);
    else if(dwRet == EINVAL)
        printf("Signal %d is invalid!\n", dwSigNo);
    else
       printf("Thread %x is alive!\n", (unsigned int)tThrdId);

    return dwRet;
}

void *WorkerThread(void *pvArg)
{
    pthread_t tThrdId = pthread_self();
    pthread_detach(tThrdId);
  
    printf("Thread %x starts to work!\n", (unsigned int)tThrdId);
    //working...
    int dwVal = 1;
    while(1)
        dwVal += 5;
}

/// 

int main(void)
{
    printf("Main thread %x is running!\n", (unsigned int)pthread_self());

    //屏蔽SIGUSR1等信号，新创建的线程将继承该屏蔽字
    sigset_t tBlockSigs;
    sigemptyset(&tBlockSigs);
    sigaddset(&tBlockSigs, SIGRTMIN);
    sigaddset(&tBlockSigs, SIGRTMIN+2);
    sigaddset(&tBlockSigs, SIGRTMAX);
    sigaddset(&tBlockSigs, SIGUSR1);
    sigaddset(&tBlockSigs, SIGUSR2);
    sigaddset(&tBlockSigs, SIGINT);
  
    sigaddset(&tBlockSigs, SIGSEGV); //试图阻塞SIGSEGV信号
  
    //设置线程信号屏蔽字
    pthread_sigmask(SIG_BLOCK, &tBlockSigs, NULL);
  
    signal(SIGINT, sighandler); //试图捕捉SIGINT信号
  
    //创建一个管理线程，该线程负责信号的同步处理
    pthread_t tMgrThrdId;
    pthread_create(&tMgrThrdId, NULL, SigMgrThread, &tBlockSigs);
    printf("Create a signal manager thread %x!\n", (unsigned int)tMgrThrdId);
    
    //创建另一个管理线程，该线程试图与tMgrThrdId对应的管理线程竞争信号
    pthread_t tMgrThrdId2;
    pthread_create(&tMgrThrdId2, NULL, SigMgrThread, &tBlockSigs);
    printf("Create another signal manager thread %x!\n", (unsigned int)tMgrThrdId2);
  
    //创建一个工作线程，该线程继承主线程(创建者)的信号屏蔽字
    pthread_t WkrThrdId;
    pthread_create(&WkrThrdId, NULL, WorkerThread, NULL);
    printf("Create a worker thread %x!\n", (unsigned int)WkrThrdId);
  
    pid_t tPid = getpid();
    
    //向进程自身发送信号，这些信号将由tMgrThrdId线程统一处理
    //信号发送时若tMgrThrdId尚未启动，则这些信号将一直阻塞
    printf("Send signals...\n");
    kill(tPid, SIGRTMAX);
    kill(tPid, SIGRTMAX);
    kill(tPid, SIGRTMIN+2);
    kill(tPid, SIGRTMIN);
    kill(tPid, SIGRTMIN+2);
    kill(tPid, SIGRTMIN);
    kill(tPid, SIGUSR2);
    kill(tPid, SIGUSR2);
    kill(tPid, SIGUSR1);
    kill(tPid, SIGUSR1);
  
    int dwRet = sleep(1000);
    printf("%d seconds left to sleep!\n", dwRet);
  
    ThreadKill(WkrThrdId, 0); //不建议向已经分离的线程发送信号
 
    sleep(1000);
    int *p=NULL; *p=0; //触发段错误(SIGSEGV)

    return 0;
 }
```

以下按行解释和分析上述执行结果：

1.相同的非实时信号(编号小于SIGRTMIN)不会在信号队列中排队，只被递送一次；相同的实时信号(编号范围为SIGRTMIN~SIGRTMAX)则会在信号队列中排队，并按照顺序全部递送。若信号队列中有多个非实时和实时信号排队，则先递送编号较小的信号，如SIGUSR1(10)先于SIGUSR2(12)，SIGRTMIN(34)先于SIGRTMAX(64)。但实际上，仅规定多个未决的实时信号中，优先递送编号最小者。而实时信号和非实时信号之间，或多个非实时信号之间，递送顺序未定义。
  
2.sigwait()函数是线程安全(thread-safe)的。但当tMgrThrdId和tMgrThrdId2同时等待信号时，只有先创建的tMgrThrdId(SigMgrThread)线程等到信号。因此，不要使用多个线程等待同一信号。

3.调用pthread_create()返回后，新创建的线程可能还未启动；反之，该函数返回前新创建线程可能已经启动。

4.SIGQUIT信号被主线程捕获，因此不会中断SigMgrThread中的sigwaitinfo()调用。虽然SIGQUIT安装(signal语句)在SigMgrThread内，由于主线程共享该处理行为，SIGQUIT信号仍将被主线程捕获。

5.sleep()函数使调用进程被挂起。当调用进程捕获某个信号时，sleep()提前返回，其返回值为未睡够时间(所要求的时间减去实际休眠时间)。注意，sigwait()等到的信号并不会导致sleep()提前返回。**因此，示例中发送SIGQUIT信号可使sleep()提前返回，而SIGINT信号不行。**

6.在线程中尽量避免使用sleep()或usleep()，而应使用nanosleep()。前者可能基于SIGALARM信号实现(易受干扰)，后者则非常安全。此外，usleep()在POSIX 2008中被废弃。

7.WorkerThread线程启动后调用pthread_detach()进入分离状态，主线程将无法得知其何时终止。示例中WorkerThread线程一直运行，故可通过ThreadKill()检查其是否存在。但需注意，这种方法并不安全。

8.已注册信号处理捕获SIGINT信号，同时又调用sigwait()等待该信号。最终后者等到该信号，可见sigwait()优先级更高。

9.sigwait()调用从未决队列中删除该信号，但并不改变信号屏蔽字。当sigwait()函数返回时，它所等待的信号仍旧被阻塞。因此，再次发送SIGINT信号时，仍被sigwait()函数等到。

10.通过pthread_sigmask()阻塞SIGSEGV信号后，sigwait()并未等到该信号。系统输出"Segmentation fault"错误后，程序直接退出。因此，不要试图阻塞或等待SIGSEGV等硬件致命错误。若按照传统异步方式使用 signal()/sigaction()注册信号处理函数进行处理，则需要跳过引发异常的指令(longjmp)或直接退出进程(exit)。注意，SIGSEGV信号发送至引起该事件的线程中。例如，若在主线程内解除对该信号的阻塞并安装处理函数sighandler()，则当SigMgrThread线程内发生段错误时，执行结果将显示该线程捕获SIGSEGV信号。


#### 例子总结
    
Linux 线程编程中，需谨记两点：1)信号处理由进程中所有线程共享；2)一个信号只能被一个线程处理。具体编程实践中，需注意以下事项：
- 不要在线程信号屏蔽字中阻塞、等待和捕获不可忽略的信号(不起作用)，如SIGKILL和SIGSTOP。
- 不要在线程中阻塞或等待SIGFPE/SIGILL/SIGSEGV/SIGBUS等硬件致命错误，而应捕获它们。
- 在创建线程前阻塞这些信号(新线程继承信号屏蔽字)，然后仅在sigwait()中隐式地解除信号集的阻塞。
- 不要在多个线程中调用sigwait()等待同一信号，应设置一个调用该函数的专用线程。
- 闹钟定时器是进程资源，且进程内所有线程共享相同的SIGALARM信号处理，故它们不可能互不干扰地使用闹钟定时器。
- 当一个线程试图唤醒另一线程时，应使用条件变量，而不是信号。


#### 引用
- [Linux线程编程之信号处理](https://www.cnblogs.com/clover-toeic/p/4126594.html)

