---
layout: post
title: linux 进程间通信-信号（signal）
date: 2014-10-11 22:02:39
categories: Linux
tags: 系统编程
excerpt: linux process communication about signal
---

### 信号（signal）

信号是UNIX和Linux系统响应某些条件而产生的一个事件，接收到该信号的进程会相应地采取一些行动。通常信号是由一个错误产生的。但它们还可以作为进程间通信或修改行为的一种方式，明确地由一个进程发送给另一个进程。一个信号的产生叫生成，接收到一个信号叫捕获。

一信号的种类

信号的名称是在头文件signal.h中定义的，信号都以SIG开头，常用的信号并不多，常用的信号如下：

```sh
[root@localhost carlos]# kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

列表中，编号为1 ~ 31的信号为传统UNIX支持的信号，是不可靠信号(非实时的)，编号为32 ~ 63的信号是后来扩充的，称做可靠信号(实时信号)。不可靠信号和可靠信号的区别在于前者不支持排队，可能会造成信号丢失，而后者不会。

二 信号的处理——signal函数

   程序可用使用signal函数来处理指定的信号，主要通过忽略和恢复其默认行为来工作。signal函数的原型如下：

```c
#include <signal.h>
void (*signal(int sig, void (*func)(int)))(int);
```

这是一个相当复杂的声明，耐心点看可以知道signal是一个带有sig和func两个参数的函数，func是一个类型为void (*)(int)的函数指针。该函数返回一个与func相同类型的指针，指向先前指定信号处理函数的函数指针。准备捕获的信号的参数由sig给出，接收到的指定信号后要调用的函数由参数func给出。其实这个函数的使用是相当简单的，通过下面的例子就可以知道。注意信号处理函数的原型必须为void func（int），或者是下面的特殊值：
SIG_IGN:忽略信号
SIG_DFL:恢复信号的默认行为
例子

```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void ouch(int sig)
{
    printf("\nOUCH! - I got signal %d\n", sig);
    //恢复终端中断信号SIGINT的默认行为
    (void) signal(SIGINT, SIG_DFL);
}

int main()
{
    //改变终端中断信号SIGINT的默认行为，使之执行ouch函数
    //而不是终止程序的执行
    (void) signal(SIGINT, ouch);
    while(1)
    {
        printf("Hello World!\n");
        sleep(1);
    }
    return 0;
}
```

结果：第一秒种输出一个 Hello World!,用ctrl+c中断后，输出OUCH! - I got signal 2 继续输出Hello World! 因为SIGINT的默认行为被signal函数改变了，当进程接受到信号SIGINT时，它就去调用函数ouch去处理。

三 信号的处理——信号处理

一个更加健壮的信号接口——sigaction函数。它的原型为：

```c 
#include <signal.h>
int sigaction(int sig, const struct sigaction *act, struct sigaction *oact);
```

参数： 
signum：除了SIGKILL和SIGSTOP之外的其它任何信号编码。
act：    如果值非NULL，将安装为sig关联信号的新处理方式。
oldact： 如果值非NULL，存储以前对sig关联信号的处理方式。
sigaction的结构形态如下：

```
struct sigaction {
    void (*sa_handler)(int);
    void (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t sa_mask;
    int sa_flags;
    void (*sa_restorer)(void);
}
```

* sa_handler和sa_sigaction共用一个联合体(union)，所以不要同时指定两个字段的值。
* sa_restorer字段已淘汰，不应该再被使用。
* sa_handler字段指定与signum信号关联的行为，可能是SIG_DFL默认行为，SIG_IGN忽略接送到的信号，或者一个信号处理函数指针。这个函数以一个信号编码作为它的唯一参数，相当于signal函数的func参数。
* 如果sa_flags中存在SA_SIGINFO标志，那么sasigaction将作为signum信号的处理函数。这个函数的第一参数是信号编码，第二参数是siginfo_t结构的指针，第三参数是ucontext_t结构指针(造型为void *)。sa_mask指定信号处理函数执行的过程中应被阻塞的信号。另外，除了SA_NODEFER标志被指定外，触发信号处理函数执行的那个信号也会被阻塞。
* sa_flags指定一系列用于修改信号处理过程行为的标志，主要有：
* SA_INTERRUPT 由此信号中断的系统调用不会自动重启
* SA_RESTART 由此信号中断的系统调用会自动重启
* SA_SIGINFO 提供附加信息，一个指向siginfo结构的指针以及一个指向进程上下文标识符的指针。

```c
#include <unistd.h>
#include <stdio.h>
#include <signal.h>
void ouch(int sig)
{
    printf("\nOUCH! - I got signal %d\n", sig);
}

int main()
{
    struct sigaction act;
    act.sa_handler = ouch;
    //创建空的信号屏蔽字，即不屏蔽任何信息
    sigemptyset(&act.sa_mask);
    //使sigaction函数重置为默认行为
    act.sa_flags = SA_RESETHAND;
    sigaction(SIGINT, &act, 0);
 
    while(1)
    {
        printf("Hello World!\n");
        sleep(1);
    }
    return 0;
}
```

运行结果与前一个例子中的相同。注意sigaction函数在默认情况下是不被重置的，如果要想它重置，则sa_flags就要为SA_RESETHAND。 
 
四 发送信号

我们可以通过两个函数kill和alarm来发送一个信号。
(1)kill函数
进程可以通过kill函数向包括它本身在内的其他进程发送一个信号。

```c
#include <sys/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
```

它的作用把信号sig发送给进程号为pid的进程，成功时返回0。
kill调用失败返回-1，调用失败通常有三大原因：
* 给定的信号无效（errno = EINVAL)
* 发送权限不够( errno = EPERM ）
*目标进程不存在( errno = ESRCH )

(2)alarm函数
这个函数跟它的名字一样，给我们提供了一个闹钟的功能，进程可以调用alarm函数在经过预定时间后向发送一个SIGALRM信号。
alarm函数的型如下：   
```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
```

alarm函数用来在seconds秒之后安排发送一个SIGALRM信号，如果seconds为0，将取消所有已设置的闹钟请求。alarm函数的返回值是以前设置的闹钟时间的余留秒数，如果返回失败返回-1。

五 信号处理函数的安全问题

有一种场景：当进程接收到一个信号时，转到你关联的函数中执行，但是在执行的时候，进程又接收到同一个信号或另一个信号，又要执行相关联的函数时，程序会怎么执行？
信号处理函数可以在其执行期间被中断并被再次调用。当返回到第一次调用时，它能否继续正确操作是很关键的。这不仅仅是递归的问题，而是可重入的（即可以完全地进入和再次执行）的问题。而反观Linux，其内核在同一时期负责处理多个设备的中断服务例程就需要可重入的，因为优先级更高的中断可能会在同一段代码的执行期间“插入”进来。
简言之，就是说，我们的信号处理函数要是可重入的，即离开后可再次安全地进入和再次执行，要使信号处理函数是可重入的，则在信息处理函数中不能调用不可重入的函数。下面给出可重入的函数在列表，不在此表中的函数都是不可重入的，可重入函数表如下：

 func       | func       | func          | func        
------------|------------|---------------|-------------
 access     | alarm      | cfgetispeed   | cfgetospeed 
 cfsetispeed| cfsetospeed| chdir         | chmod       
 dup        | execle     | execve        | dup2        
 fcntl      | fork       | fstat         | getegid     
 getppid    | getuid     | kill          | link        
 geteuid    | lseek      | mkdir         | mkfifo      
 open       | pathconf   | pause         | pipe        
 setpgid    | setsid     | setuid        | sigaction   
 sigismember| signal     | sigpending    | sigprocmask 
 sigsuspend | sleep      | stat          | sysconf     
 tcdrain    | tcflow     | tcflush       | tcgetattr   
 tcgetpgrp  | tcsendbreak| tcsetattr     | tcsetpgrp   
 time       | times      | umask         | uname       
 unlink     | utime      | wait          | waitpid     
 write      |            |               |             

不可重入的函数:
（a） 已知它们使用静态数据结构，
（b） 它们调用malloc或free
（c）标准I/O函数


一篇很不错的文章：
[Linux信号（signal) 机制分析][1]

[1]: http://www.cnblogs.com/hoys/archive/2012/08/19/2646377.html       "Linux信号（signal) 机制分析" 

















