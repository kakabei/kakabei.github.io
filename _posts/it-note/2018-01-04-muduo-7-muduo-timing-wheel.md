---
layout: post
title: muduo笔记 第七章 时间轮
date: 2018-01-04 20:21:12
categories: muduo
tags:  技术学习笔记 系统编程 muduo  
excerpt: 时间轮
---


用timing wheel 踢掉空闲连接。

#### 如果一个连接连续几秒内没有收到数据，就把它断开，为此有两种简单、粗暴的做法：

1.每个连接保存"最后收到数据的时间lastReceiveTime"， 然后用一个定时器，每秒遍历一遍所有连接， 断开那些(now - connection. lastReceiveTime) > 8s 的connection。 这种做法全局只有一个repeated timer， 不过每次timeout 都要检查全部连接，如果连接数目比较大（几千上万） 这一步可能会比较费时。 ·

2.每个连接设置一个one-shot timer， 超时定为8s， 在超时的时候就断开本连接。当然，每次收到数据要去更新timer。 这种做法需要很多个one-shot timer， 会频繁地更新timers。如果连接数目比较大， 可能对EventLoop 的 TimerQueue 造成 压力。

连接超时不需要精确定时，只要大致8秒超时断开就行，多一秒、少一秒关系不大。

处理连接超时可用一个简单的数据结构：8个桶组成的循环队列。

第1个桶放1秒之后将要超时的连接，第2个桶放2秒之后将要超时的连接。每个连接一收到数据就把自己放到第8个桶，然后在每秒的timer里把第一个桶里的连接断开，把这个空桶挪到队尾。

这样大致可以做到8秒没有数据就超时断开连接。

#### 时间轮的原理

简单的时间轮的基本结构是一个循环队列，还有一个指向队尾的指针（tail）。这个指针每秒移动一格，就像钟表上的时针。

以下是某一时刻timing wheel 的状态（ 见图7-42的左图）， 格子里的数字是倒计时（ 与通常的 timing wheel 相反）， 表示这个格子（ 桶子） 中连接的 剩余寿命。1秒以后（见图 7-42的右图）， tail 指针移动一格， 原来四点钟方向的格子被清空，其中的连接已被断开。

![](/assets/muduo/7-time-wheel1.png) 

连接超时时被踢掉的过程
假设在某个时刻，conn1到达， 把它放到当前格子中，它的剩余寿命是7秒（见图7-43的左图）。 此后conn1 上没有收到数据。 1秒之后（见图7-43的右图），tail指向下一个格子， conn 1的剩余寿命是6秒。

![](/assets/muduo/7-time-wheel2.png) 

又 过了几 秒， tail指向conn1之前的那个格子， conn1即将被断开（见图7-44的左图）。下一秒（见图7-44的右图），tail重新指向conn1原来所在的格子，清空其中的数据，断开conn1连接。
![](/assets/muduo/7-time-wheel3.png) 

#### 连接刷新

如果在断开conn1之前收到数据，就把它移到当前的格子里。conn1的剩余寿命是3秒（见图7-45的左图），此时 conn 1收到数据，它的寿命恢复为7秒（见图 7-45 的右图）。

![](/assets/muduo/7-time-wheel4.png) 

时间继续前进，conn1寿命递减，不过它已经比第一种情况长寿了（见图7-46）。

![](/assets/muduo/7-time-wheel5.png) 

#### 多个连接

timingwheel中的每个格子是个hashset，可以容纳不止一个连接。比如一开始，conn1到达。随后，conn2到达（见图7-47），这时候tail还没有移动，两个连接位于同一个格子中，具有相同的剩余寿命。（在图7-47中画成链表，代码中是哈希表。）

---

代码的实现体现在主要在三个地方。

1）新的连接上来时。

![](/assets/muduo/7-time-wheel6.png) 

entry 为 shared_ptr类型。连接时会把entery插入到循环队列中的set当中。

2）当有数的数据过来时。

![](/assets/muduo/7-time-wheel7.png) 

也会把这个连接的entry加入到循环队列中的set当中。

3）定时器
每一秒钟会从循环队列中拿掉一个Set。entry的shared_ptr计数就会减1。

![](/assets/muduo/7-time-wheel8.png) 

当entry的shared_ptr被减到0时，entry就会被释放。就会断开连接。


---

 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.
