---
layout: post
title: libevent 初步分析
date: 2016-07-15 17:32:03
categories: Linux
tags: libevent network 
excerpt: network frame libevent
---

### 一 reactor模式

整个libevent本身就是一个reactor。reactor翻译成反应堆，是一种事件驱动机制。libevent，底层所运用的就是例epoll这样的一些模型。

应用程序一般都要在reactor注册回调函数。当事件被触发时，回调函数会被调用。这些事件可能是I/O读写， 定时器和信号。

reactor模型：必备的几个组件：事件源、reactor框架、多路利用机制和事件处理函数。
	
**事件源：**

Linux上是文件描述符， win下的socket 或 handle.

**event demultipexer (事件多路分发机制)：**

linux下如：epoll、 select、kqueue、devpoll

当有事件到达时， event demultiplexer会发出通知。这时相关的事件就成了就绪状态。libevent会在非阻塞的情况下进行处理。

libevent用 eventtop对 select epoll poll 等进行了封闭，形成统一的接口。

**reactor 反应器：**

reactor是事件管理接口。内部使用 event demultiplexer 注册、注销事件；并运行事件循环，
当有事件进入“就绪”状态时，调用注册事件的回调函数处理事件。对应到 libevent 中，就是 event_base 结构体。

**事件处理流程图：**

1）首先应用程序准备并初始化 event，设置好事件类型和回调函数；

2）向 libevent 添加该事件 event。对于定时事件， libevent 使用一个小根堆管理， key 为超
时时间；对于 Signal 和 I/O 事件， libevent 将其放入到等待链表（ wait list）中，这是一
个双向链表结构；

3） 程序调用 event_base_dispatch()系列函数进入无限循环，等待事件，以 select()函数为例；
每次循环前 libevent 会检查定时事件的最小超时时间 tv，根据 tv 设置 select()的最大等
待时间，以便于后面及时处理超时事件；当 select()返回后，首先检查超时事件，然后检查 I/O 事件；

![](/assets/network/libevent_event_next.png)  

### 二  源码文件组织结构

头文件、内部使用的头文件、辅助功能函数、日志、libevent框架、对系统I/O多路复用机制的封装
信号管理、定时事件管理、缓冲区管理、基本数据和基于libevent的两个实用库的向个部分。

**头文件**

event.h：事件宏定义、接口函数声明，主要结构体event的声明；

xxx-internal.h：内部数据结构和函数，对外不可见，以达到信息隐藏的目的；
	
**libevent框架**

event.c：event整体框架的代码实现；
	
对系统I/O多路复用机制的封装

- epoll.c：对epoll的封装；
- select.c：对select的封装；
- devpoll.c：对dev/poll的封装;
- kqueue.c：对kqueue的封装；

**定时事件管理**

min-heap.h：其实就是一个以时间作为key的小根堆结构；

**信号管理**

signal.c：对信号事件的处理；
	
**辅助功能函数**

evutil.h 和evutil.c：一些辅助功能函数，包括创建socket pair和一些时间操作函数：加、减和比较等。

**日志**

log.h和log.c：log日志函数

**缓冲区管理**

evbuffer.c和buffer.c：libevent对缓冲区的封装；

**基本数据结构**

compat\sys下的两个源文件：queue.h是libevent基本数据结构的实现，包括链表，双向链表，队列等；
_libevent_time.h：一些用于时间操作的结构体定义、函数和宏定义；
	
**实用网络库**

http和evdns：是基于libevent实现的http服务器和异步dns查询库


### 三 事件event
	
主要的结构体：

```c
struct event_base // event最主要的结构，
struct eventop    // 定义backend结构， 通过它定义各个模型，
```

- ev_events： event关注的事件类型，它可以是以下3种类型：
- I/O事件：  EV_WRITE和EV_READ
- 定时事件： EV_TIMEOUT
- 信号：     EV_SIGNAL	
- 辅助选项： EV_PERSIST，表明是一个永久事件

ev_next， ev_active_next 和 ev_signal_next 都是双向链表节点指针。

I/O和Signal事件使用了双向链表。

定时事件 使用了小根堆 min_heap_idx.

ev_next 是该I/O事件在链表中的位置，表示是“已注册事件链表”。
ev_signal_next signal事件在signal事件链表中的位置。
ev_active_next libevent将所有的激活事件放入到链表active list中，然后遍历 active list执行调度，ev_active_next就指明了event在active list中的位置。
 
***libevent 对 event 的管理***

![](/assets/network/libevent_event_managemant.png)  

事件设置的接口函数 

libevent 提供了函数：event_set(), event_base_set(), event_priority_set()。

设置事件 如：I/O事件、 时间事件、信号事件:

```c
void event_set(struct event *ev, int fd, short events,
		void (*callback)(int, short, void *), void *arg)
```

设置 event ev 将要注册到的 event_base；

```c
int event_base_set(struct event_base *base, struct event *ev)
```

设置event ev的优先级:

 ```c  
int event_priority_set(struct event *ev, int pri)
```

### 四 事件处理框架

事件处理都是围绕着 event_base。

初始化一个 event_base。 本质上是调用了 event_base_new_with_config。

```c
struct event_base * event_init()
```

也是 初始化一个 event_base。不同的是 先创建了一个struct event_config。 这个东西是干什么用的还不清楚。

```c
struct event_base * event_base_new()
```

内部主要调用了 event_base_new_with_config。()

```c
int event_add(struct event *ev, const struct timeval *timeout);
```

内部主要调用了 event_del_internal()

```c
int event_del(struct event *ev);
```
函数将删除事件 ev，对于 I/O 事件，从 I/O 的 demultiplexer 上将事件注销；对于 Signal
事件，将从 Signal 事件链表中删除；对于定时事件，将从堆上删除。

```c
int event_base_loop(struct event_base *base, int loops);

void event_active(struct event *event, int res, short events);

void event_process_active(struct event_base *base);
```

event_base_loop： 等待事件，分发事件。

event_active：就绪事件。

event_process_active：　处理就绪事件。

### 五  事件主循环

struct evsig_info // 这个又是干吗的？

**I/O和Timer事件的统一**
	
libevent将Timer和Signal事件都统一到了系统的I/O 的demultiplex机制中了，

堆是一种经典的数据结构，向堆中插入、删除元素时间复杂度都是O(lgN)。而获取最小key值（小根堆）的复杂度为O(1)。


**I/O和Signal事件的统一**

如果当Signal发生时，并不立即调用event的callback函数处理信号，而是设法通知系统的I/O机制，让其返回，然后再统一和I/O事件以及Timer一起处理。


### 六 集成信号处理

singal和I/O的事件统一是通过 socket pair的方式实现。（这个方式有点像是管道）

### 七 I/O多路复用技术

libevent根据系统配置和编译选项决定使用哪一种I/O demultiplex机制，而不支持在运行阶段根据配置再次选择。


### 十一 时间管理 

Libevent 本身不是多线程安全的
libevent 库的其他组件提供其他功能，包括缓冲的事件系统（用于缓冲发送到客户端/从客户端接收的数据）以及 HTTP、DNS 和 RPC 系统的核心实现。

可以对比一下 libev 

---

参考：[**libevent源码深度剖析.pdf**](http://pan.baidu.com/s/1hssU5KC)
	