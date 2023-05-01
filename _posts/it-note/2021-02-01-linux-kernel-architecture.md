---
layout: post
title: Linux 内核体系结构
date: 2023-04-23 10:12:15
categories: operating-system
tags: operating-system 技术学习笔记
excerpt:  Linux 内核的编制模式和体系结构
---
 
 一个完整可用的操作系统主要由4部分组成： 硬件、操作系统内核、操作系统服务和应用程序。 如下图：

 操作系统内核主要用于对硬件资料的抽象和访问高度。 操作系统内核的主要用途就是为了与计算机硬件交互，实现硬件对外的接口，调度对硬件资源的访问。

![](/assets/operating-system/linux-kernel-2023-04-29-17-31-54.png) 


#　Linux 内核模式

操作系统内核的结构模式可以分为单内核模式和层次式微内核模式。 

Linux 0.11 内核，是采用了单核模式。 这种模式的优点是： 代码结构紧凑、执行速度快；缺点是：层次结构不强。

单内核模式的大致可以分为三个层次： 调用服务的主程序层、 执行系统调用的服务层、支持系统调用的底层函数。 

流程大致为： 应用程序执行系统调用指令（int x80）， CPU 从用户态切换到内核态， 然后调用特定的系统调用服务程序， 这个系统调用服务程序调用底层的函数， 完成指定的功能后切回用户态， 返回给应用程序。 

![](/assets/operating-system/linux-kernel-2023-04-29-17-46-28.png)

#  Linux 内核系统体系结构

Linux 内核主要由5个模块构成： 进程调度模块、内存管理模块、文件系统模块、进程间通信模块和网络接口模块。 

进程调度模块负责控制进程对CPU 资源的使用。 调度的策略是各进程公平合理使用 CPU。 

内存管理模块确保所有进程安全的共享机器主内存区，也支持虚拟内存管理方式。并利用文件系统把暂时不用的内存数据交换到外部存储设备上去。

文件系统模块支持对外部设备的驱动和存储。

进程间通信模块子系统支持多进程间的信息交换方式。

网络接口模块负责多种网络通信标准的访问和网络硬件。 

之间的关系如下：虚线和虚框部分表示 Linux 0.11 中还未实现的部分。

![](/assets/operating-system/linux-kernel-2023-04-29-17-58-38.png)


所有的模块都与进程调度模块存在关系。它们都需要依靠进程调度程序来挂起或重新运行它们的进程。

进程调度子系统需要使用内存管理器来调整一特定进程所使用的物理内存空间。

进程间通信子系统则需要依靠内存管理器来支持共享内存通信机制

根据Linux 0.11 的源码结构，内核主要模块如下：

![](/assets/operating-system/linux-kernel-2023-04-29_23-13-18.png)

粗线方框分别对应内核源代码的目录组织结构。


# 中断机制

中断机制，我理解起来有些难度，后面要好好研究一下。 

>对于 Linux 内核来说，中断信号通常分为两类：硬件中断和软件中断(异常)。

每个中断是由 `0-255` 之间的一个数字来标识。

中断 `int0--int31(0x00--0x1f)` 属于软件中断，Intel 公司称之为异常。

中断 `int32--int255 (0x20--0xff)` 可以由用户自己设定。

则将 `int32--int47(0x20--0x2f)` 对应于 8259A 中断控制芯片发出的硬件中断请求信号 IRQ0-IRQ15，

并把程序编程发出的系统调用(system_call)中断设置为 int128(0x80)。

# 系统定时

Linux 0.11 内核中，可编程定时芯片被设置成每隔10毫秒就发出一个时钟中断（IRQ0）信号。这个时间节拍就是系统运行的脉搏，我们称之为 1 个**系统滴答**。

# Linux 进程控制

利用分时技术，在 Linux 操作系统上同时可以运行多个进程。分时技术的基本原理是把 CPU 的运行时间划分成一个个规定长度的
时间片，让每个进程在一个时间片内运行。

对于 Linux 0.11 内核来讲，系统最多可有 64 个进程同时存在，一个进程可以在内核态（kernel mode）或用户态（user mode）下执行，因此，Linux
内核堆栈和用户堆栈是分开的。

内核程序是通过进程表以进程进行管理。每一个进程在进程表中占有一项。进程表项是一个 `taks_struct` 任务结构指针。 定义在 `include/linux/sched.h` 

也叫 进程控制块 PCB（Process Control Block）或进程描述符 PD（Processor Descriptor）。

Linux 0.11 的 任务数据结构 如下:

```c

struct task_struct {
/* these are hardcoded - don't touch */
	long state;	/* -1 unrunnable, 0 runnable, >0 stopped */
	long counter;
	long priority;
	long signal;
	struct sigaction sigaction[32];
	long blocked;	/* bitmap of masked signals */
/* various fields */
	int exit_code;
	unsigned long start_code,end_code,end_data,brk,start_stack;
	long pid,father,pgrp,session,leader;
	unsigned short uid,euid,suid;
	unsigned short gid,egid,sgid;
	long alarm;
	long utime,stime,cutime,cstime,start_time;
	unsigned short used_math;
/* file system info */
	int tty;		/* -1 if no tty, so it must be signed */
	unsigned short umask;
	struct m_inode * pwd;
	struct m_inode * root;
	struct m_inode * executable;
	unsigned long close_on_exec;
	struct file * filp[NR_OPEN];
/* ldt for this task 0 - zero 1 - cs 2 - ds&ss */
	struct desc_struct ldt[3];
/* tss for this task */
	struct tss_struct tss;
};
```

CPU 的所有寄存器中的值、进程的状态以及堆栈中的内容被称为该 **进程的上下文**。

当内核需要切换（switch）至另一个进程时, 会把当前进程的上下文均保存在进程的任务数据结构中。

 一个进程在其生存期内，可处于一组不同的状态下，称为进程状态。保存在进程任务结构的 state 字段中。

![](/assets/operating-system/linux-kernel-_2023-05-01-16-41-11.png)

睡眠等待状态被分为可中断的和不可中断的等待状态

**可中断睡眠状态（TASK_INTERRUPTIBLE）**

当进程处于可中断等待状态时，系统不会调度该进行执行。当系统产生一个中断或者释放了进程正在等待的资源，或者进程收到一个信号，都可以唤醒进程转换到就绪状态（运行状态）。

**不可中断睡眠状态（TASK_UNINTERRUPTIBLE）**

与可中断睡眠状态类似。但处于该状态的进程只有被使用 wake_up()函数明确唤醒时才能转换到可运行的就绪状态。

当一个进程的运行时间片用完，系统就会使用调度程序强制切换到其它的进程去执行。另外，如果
进程在内核态执行时需要等待系统的某个资源，此时该进程就会调用 sleep_on()或 sleep_on_interruptible()
自愿地放弃 CPU 的使用权，而让调度程序去执行其它进程。进程则进入睡眠状态（TASK_UNINTERRUPTIBLE 或 TASK_INTERRUPTIBLE）。

进程初始化

- 引导程序把内核从磁盘上加载到内存中。 
- 系统进入保护模式下
- 行系统初始化程序 `init/main.c`
    - 确定如何分配使用系统物理内存
    - 调用内核各部分的初始化函数分别对内存管理、中断处理、块设备和字符设备、进程管理以及硬盘和软盘硬件进行初始化处
- 设置自己成为任务 0 的进程，并使用 fork() 调用前次创建出进程 1。
- 进程 1 中程序将继续进行应用环境的初始化并执行 shell 登录程序。
- 原进程 0 则会在系统空闲时被调度执行，此时任务 0 仅执行 pause()系统调用，并又会调用调度函数。


 ---

 1、《linux 内核完全注释》 by 赵烔