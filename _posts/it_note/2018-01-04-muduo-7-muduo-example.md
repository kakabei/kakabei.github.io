---
layout: post
title: muduo笔记 第七章 muduo编程示例
date: 2018-01-04 20:18:12
categories: Linux muduo
tags:  技术阅读笔记 系统编程 muduo笔记 tcp network thread
excerpt: muduo编程示例
---


#### 为什么TcpConnection::shutdown()没有直接关闭TCP连接

muduo TcpConnection没有提供close()，而只提供shutdown()，这么做是为了收发数据的完整性。

也就是说muduo把"主动关闭连接"这件事情分成两步来做，如果要主动关闭连接，它会先关本地"写"端，等对方关闭之后，再关本地"读"端。

就是要把buff的数据全部发完之后才会关闭它。

#### TCP分包
在TCP这种字节流协议上做应用层分包是网络编程的基本需求。

分包指的是在发生一个消息（message）或一帧（frame）数据时，通过一定的处理，让接收方能从字节流中识别并截取（还原）出一个个消息。


#### muduo buff类的设计与使用

event loop是non-blocking网络编程的核心，在现实生活中，non-blocking几乎总是和IO multiplexing一起使用，

原因有两点： 

1.没有人真的会用轮询（busy-pooling）来检查某个non-blocking IO操作是否完成，这样太浪费CPU cycles。 


2.IO multiplexing一般不能和blocking IO用在一起，因为blocking IO中`read()/write()/accept()/connect()`都有可能阻塞当前线程，这样线程就没办法处理其他socket上的IO事件了。

#### 为什么non-blocking网络编程中应用层buffer是必需的

non-blocking IO的核心思想是避免阻塞在read()或write()或其他IO系统调用上，这样可以最大限度地复用thread-of-control，让一个线程能服务于多个socket连接。

IO线程只能阻塞在IO multiplexing函数上，如select/poll/epoll_wait。这样一来，应用层的缓冲是必需的，每个TCP socket都要有stateful的input buffer和output buffer。

#### muduo Buffer的设计要点： 

1.对外表现为一块连续的内存(char* p, int len)，以方便客户代码的编写。 

2.其size()可以自动增长，以适应不同大小的消息。它不是一个fixed size array（例如char buf[8192]）。 

3.内部以`std::vector<char>`来保存数据，并提供相应的访问函数。 Buffer其实像是一个queue，从末尾写入数据，从头部读出数据。 谁会用Buffer？谁写谁读？根据前文分析，TcpConnection会有两个Buffer成员，input buffer与output buffer。 

4.input buffer，TcpConnection会从socket读取数据，然后写入input buffer（其实这一步是用`Buffer::readFd()`完成的）；客户代码从input buffer读取数据。 output buffer，客户代码会把数据写入output buffer（其实这一步是用`TcpConnection::send()`完成的）；TcpConnection从output buffer读取数据并写入socket。

5.其实，input和output是针对客户代码而言的，客户代码从input读，往output写。TcpConnection的读写正好相反。

#### protobuf从消息名创建消息，然后分发给不同的处理函数。

#### 限制并发连接数

一方面，我们不希望服务程序超载；另一方面，更因为filedescriptor是稀缺资源，如果出现filedescriptor耗尽，很棘手，跟“malloc()失败/new抛出std::bad_alloc"差不多同样棘手。
    
在服务端中如果在accetp中出现fd耗尽，那么就可能无法及时通知客户关闭。

所以，可以准备一个空闲的文件描述符。遇到这种情况，先关闭这个空闲文件，获得一个文件描述符的名额；再accept(2)拿到新socket连接的描述符；随后立刻close(2)它，这样就优雅地断开了客户端连接；最后重新打开一个空闲文件，把“坑"占住，以备再次出现这种情况时使用。

另一种方式，要在onConnection时统计活着的链接数。如果超过最大就shutdown。

#### 定时器

在一般的服务端程序设计中，与时间有关的常见任务有： 

1．获取当前时间，计算时间间隔。

2．时区转换与日期计算；把纽约当地时间转换为上海当地时间；2011-02-05之后第100天是几月几号星期几；等等。 

3．定时操作，比如在预定的时间执行任务，或者在一段延时之后执行任务。

#### linux时间函数

Linux的计时函数，用于获得当前时间： 

- time(2) / time_t（秒） 
- ftime(3) / struct timeb（毫秒） 
- gettimeofday(2) / struct timeval（微秒）
- clock_gettime(2) / struct timespec（纳秒） 

还有gmtime / localtime / timegm / mktime / strftime / struct tm等与当前时间无关的时间格式转换函数。 
定时函数，用于让程序等待一段时间或安排计划任务： 

- sleep(3) 
- alarm(2) 
- usleep(3) 
- nanosleep(2) 
- clock_nanosleep(2) 
- getitimer(2) / setitimer(2) 
- timer_create(2) / timer_settime(2) / timer_gettime(2) / timer_delete(2) ·timerfd_create(2) / timerfd_gettime(2) / timerfd_settime(2) 

#### 我的取舍如下：
 - （计时）只使用gettimeofday(2)来获取当前时间。 
 - （定时）只使用timerfd_*系列函数来处理定时任务。


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.


