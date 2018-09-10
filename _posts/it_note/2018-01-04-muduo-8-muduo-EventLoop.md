---
layout: post
title: muduo笔记  第八章 muduo网络库设计与实现
date: 2018-01-04 20:21:12
categories: Linux muduo
tags: 系统编程 muduo tcp network thread
excerpt: muduo网络库设计与实现
---



runInLoop() 线程安全的理解

![](/assets/muduo/8-muduo-EventLoop.png) 

在线程T1 muduo::EventLoop loop;， 并loop.loop();那么这个事件应该在线程T1上跑。但是线程T2做了一件事，就是loop::runInLoop(cb); 这个时候添加回调时和loop并不是同一个线程。会有问题。

所以,runInLoop()的做法是：先判断是不是同一个线程，是的话就直接被执行，不是的话就加到挂起队列中。

// 是否在当线线程，不在就加入队列。 因为可能是其他线程执行这个代码，

```c++
void EventLoop::runInLoop(const Functor &cb)
{
  if (isInLoopThread())
  {
    cb();
  }
  else
  {
    queueInLoop(cb);
  }
}
```

在queueInLoop中会唤醒T1线程。做法是：

T1和T2线程有一个channel,写入一个字符。触发T1的事件，T1在loop中会执行被挂起的函数队列。


addTimer 要做到线程安全，就是把它的回调加入到runInLoop，这样它可以触发它所在的线程去执行。

```c++
TimerId TimerQueue::addTimer(const TimerCallback& cb,
                             Timestamp when,
                             double interval)
{
  Timer* timer = new Timer(cb, when, interval);
  loop_->runInLoop(
      boost::bind(&TimerQueue::addTimerInLoop, this, timer));
  return TimerId(timer);
}
```



#### 8.4 实现TCP 网络库

这一节主要是讲 Acceptor class。


Acceptor class是用于封装accept接受新连接的。为什么在单独成为一个模块类呢？

1.Acceptor  的成员有socket channel 。其中socket 是用了RAII handle 。

2.Channel用于观察此socket上的readable事件，并回调Acceptor:: handleRead()，后者会调用accept(2)来接受新连接，并回调用户callback。

3.关于，如果系统的fd耗尽的问题。 在一个开始，就打一个空闲的fd。当系统耗尽时，会先关闭这个空闲的fd。要给新上来的客户端accept。最后，
断开客户端，把fd交还给空闲的占用。 这么做是为了解决在系统耗尽fd时，不会断开客户端上来的链接。


#### 8.5 TcpServer  接受新连接
---
主要是讲TcpServer class。 tcp服务是管理accept获得TcpConnection。
这是一个新连接的接受的过程：

![](/assets/muduo/8-muduo-tcpserver-class.png) 

TcpServer 很简单，用户只 需要设置好callback，再调用start()。就OK了。









