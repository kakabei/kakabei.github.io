---
layout: post
title: muduo笔记 第六章 性能评测
date: 2018-01-04 20:08:12
categories: muduo
tags:  技术学习笔记 系统编程 muduo 
excerpt: 性能评测
---


#### 6.5 性能评测

性能对比原则：采用对方的性能测试方案，用muduo实现功能相同或类似的程序，然后放到相同的软硬件环境中对比。

注意这里的测试只是简单地比较了平均值；其实在严肃的性能对比中至少还应该考虑分布和百分位数（percentile）的值。限于篇幅，此处从略。

简单地说， ping pong协议是客户端和服务器都实现echo协议。 

当TCP连接建立时， 客户端向服务器发送一些数据， 服务器会echo回这些数据， 然后客户端再echo回服务器。这些数据就会像乒乓球一样在客户端和服务器之间来回传送， 直到有一方断开连接为止。 这是用来测试吞吐量的常用办法。

##### [注]

1.书中这里做了 asio libevent2和 muduo的对比。对于asio我没有接触过，那我就只关注  libevent2和muduo就可以了。

2.做对比测试时，要注明测试环境： 硬件： CPU 几核主频多少。内存多少。软件： 操作系统，版本 内核版本。(主要测框架的性能，固定这一些定量。才能做对比)
   
##### 测试方法： 分了单线程测试，和多线程测试。 多线程测试要注意测试的cpu核数。


现在的CPU很快，即便是单线程单TCP连接也能把千兆以太网的带宽跑满。如果用两台机器，所有的吞吐量测试结果都将是110MiB/s，失去了对比的意义。（用Python也能跑出同样的吞吐量，或许可以对比哪个库占的CPU少。）

可能是由于路由器或交换机的影响，对带宽有所限制。

这是测试代码 ：[https://gist.github.com/chenshuo/564985](https://gist.github.com/chenshuo/564985)

单纯程测试结果是 muduo 比 libevent2快70%

跟踪libevent2的源代码发现，它每次最多从socket读取4096字节的数据（证据在buffer.c的evbuffer_ read()函数），怪不得吞吐量比muduo小很多。

因为在这一测试中，muduo每次读取16384字节，系统调用的性价比较高。

但是把缓冲区都测试成为4K，结果，muduo还是比libevent2快18%以上。

##### [注]

1.这里指的 单线程，缓冲区大小一样。结果还是比libevent2快18%以上。那么问题在什么地方呢？

2.libevent2的多线程方式并不是很好用，可能是因为这个原因，他没有做libeven2的多线程方面测试。
 
有人会说，libevent2并不是为高吞吐量的应用场景而设计的，这样的比较不公平，胜之不武。

##### [注]
1.那是它是专用为什么目的设计的呢？

---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.