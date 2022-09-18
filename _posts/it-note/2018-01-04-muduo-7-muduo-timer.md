---
layout: post
title: muduo笔记 第七章 定时器
date: 2018-01-04 20:20:12
categories: muduo
tags:  技术阅读笔记 系统编程 muduo tcp network thread
excerpt: 定时器
---


在一般的服务端程序设计 中， 与时间有关的常见任务有： 

1、获取当前时间， 计算时间间隔。 

2、时区转换与日期计算； 把纽约当地时间转换为上海当地时间； 2011-02-05 之后第100天是几月几号星期几；等等。 

3、 定时操作， 比如在预定的时间执行任务， 或者在一段延时之后执行任务。

计时，只使用gettimeofday(2)来获取当前时间

定时，只使用timerfd_*系统函数来处理定时任务

1、 time(2) 的精度太低了， ftime(3)被废弃。 clock_gettime(2)精度最高，但是系统调用的开销比gettimeofday大。

2、gettimeofday 不是系统调用，而是在用户态实现的。 没有上下文切换和陷入内核的开销。

3、timerfd_create(2)把时间变成了一个文件描述符，该“文件”在定时器超时的那一该变得可读，这样就能很方便的融入select(2)/poll(2)框架中，用统一的方式 来处理IO事件和超时事件，这也正是Reactor模式的长处。

4、传统的Reactor利用select(2)/poll(2)/epoll(4)的timeout来实现定时功能，但poll(2)和epoll_wait(2)的定时精度只有毫秒，远低于timerfd_ settime(2)的定时精度。

#### muoduo 的定时器接口有三个。都在EventLoop中。
1.runAt 在 指定的时间调用TimerCallback；

2.runAfter 等一段时间调用TimerCallback； 

3.runEvery 以固定的间隔反复调用TimerCallback； 

 cancel 取消timer。 回调函数在EventLoop 对象所属的线程发生，与onMessage()、 onConnection() 等网络事件函数在同一个线程。

### [注] 
1、现在所在游戏项目中所用的时间函数是clock_gettime 然后再转成毫秒。

---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.