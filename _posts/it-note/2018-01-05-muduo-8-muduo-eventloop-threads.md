---
layout: post
title: muduo笔记  第八章 EventLoop 在保证线程安全所做事情
date: 2018-01-05 15:21:12
categories: muduo
tags:  系统编程 muduo 技术阅读笔记 tcp network thread
excerpt: muduo EventLoop 在保证线程安全所做事情
---


本来 一个线程一个EventLoop。很理想的状态下当然最好了。
但是一个EventLoop对象还是可能被不同的线程执行。这就导致了可能出现，本不属于该线程的事务被它执行到。出现线程同步的问题。

所以，EventLoop 用一种方式 	：runInLoop()和queueInLoop()

```c++
/// 线程安全，如果不是在当前线程，则被挂起，然后通知线程执行
/// 这个意思就是这个Functor要被在它所属的线程调用。
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

如果是在当前线程就直接执行即可。

如果不是就加放到队列中去。

```c++
std::vector<Functor> pendingFunctors_; // @GuardedBy mutex_ 这个成员会暴露给其他线程所以要 mutex_做保护。

// 在线程中回调一个函数，线程安全
void EventLoop::queueInLoop(const Functor &cb)
{
  {
    // 回调函数之所要被挂, 因为是要被其他线程执行的。所以要加锁
    MutexLockGuard lock(mutex_); 
    pendingFunctors_.push_back(cb);
  }
  // 如果在IO线程调用 queueInLoop()， 而此时正在调用 pending functor， 那么也必须唤醒。
  if (!isInLoopThread() || callingPendingFunctors_) 
  {
    wakeup();
  }
}
```

queueInLoop有一个锁， 这个锁就是负责 pendingFunctors_的。也只负责pendingFunctors_。 

然后唤醒对应的线程。 怎么做到呢？　（如果同同一线程的情况下，可能就被直接执行了。）

看一下 wakeup()

```c++
/// 往wakeupFd_写入一个uint64_t数据
void EventLoop::wakeup()
{
  uint64_t one = 1;
  ssize_t n = sockets::write(wakeupFd_, &one, sizeof one);
  if (n != sizeof one)
  {
    LOG_ERROR << "EventLoop::wakeup() writes " << n << " bytes instead of 8";
  }
}
```

wakeupFd_ 是什么 ？  在结造函数中是这样的  wakeupFd_(createEventfd()),

```c++
int createEventfd()
{
  int evtfd = ::eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);
  if (evtfd < 0)
  {
    LOG_SYSERR << "Failed in eventfd";
    abort();
  }
  return evtfd;
}

```
用的是 eventfd 事件。

eventfd类似于管道的概念，可以实现线程间的事件通知，类似于pipe。而eventfd 是一个比 pipe 更高效的线程间事件通知机制，一方面它比 pipe 少用一个 file 
descriper，节省了资源；

另一方面，eventfd 的缓冲区管理也简单得多，全部“buffer”一共只有8字节，不像pipe那样可能有不定长的真正buffer。

eventfd的缓冲区大小是sizeof(uint64_t)也就是8字节，它是一个64位的计数器，写入递增计数器，读取将得到计数器的值，并且清零。

就是说，线程间的通知机制用的是eventfd 事件。

唤醒其实就是往这个事件的fd写入一个uint64_t。

而每一个EventLoop 都会注册对这个wakeupFd_ 的读事件。即构造函数中的 wakeupChannel_(new Channel(this, wakeupFd_)),

```c++
 wakeupChannel_->setReadCallback(
      boost::bind(&EventLoop::handleRead, this)); // 设置读的回调，用于被唤醒时
  // we are always reading the wakeupfd
  wakeupChannel_->enableReading();
```

当wakeupFd_ 有写入数据时。Channel是会被触发读事件的  poller_->poll。这时唤醒所以当有了读事件时 EventLoop::handleRead 被执行。

然后会执行下面的 doPendingFunctors()

一开始想 在主循环 void EventLoop::loop() 中while的循环下 加入 doPendingFunctors()

```c++
  while (!quit_)
  {
    activeChannels_.clear();                                        // 被触发的活跃事件
    pollReturnTime_ = poller_->poll(kPollTimeMs, &activeChannels_); // activeChannels返回激活的事件
    ++iteration_;
    if (Logger::logLevel() <= Logger::TRACE)
    {
      printActiveChannels(); //打印出所有事件
    }
    // TODO sort channel by priority
    eventHandling_ = true;
    for (ChannelList::iterator it = activeChannels_.begin();
         it != activeChannels_.end(); ++it)
    {
      currentActiveChannel_ = *it;
      currentActiveChannel_->handleEvent(pollReturnTime_); //处理事件
    }
    currentActiveChannel_ = NULL;
    eventHandling_ = false;
    doPendingFunctors(); //处理被挂起的回调函数，
  }
```

那么 doPendingFunctors() 不就一直被执行了吗？ 

其实，不是这样子的。 因为 poller_->poll(); 在没有事件的情况下是会被阻塞的。 所以才要用eventfd做器事件通知。写入一个int64_t。这个时候才能唤醒  poller_->poll(); 

```c++
/*
回过头来看，muduo在处理多线程加锁访问共享数据的策略上，有一个很重要的原则:拼命减少临界区的长度
试想一下，如果没有pendingFunctors_这个数据成员，我们要想往TimerQueue中添加timer，
肯定要对TimerQueue里面的insert函数加锁，造成锁的争用，
而pendingFunctors_这个成员将锁的范围减少到了一个vector的push_back操作上。
此外，在doPendingFunctors中，利用一个栈对象减少临界区，也是很巧妙的一个重要技巧。
*/

void EventLoop::doPendingFunctors()
{
  std::vector<Functor> functors;
  callingPendingFunctors_ = true;  
  
  // 减少了临界区的长度，其它线程调用queueInLoop对pendingFunctors加锁时，就不会被阻塞
  // 避免了死锁，可能在functors里面也会调用queueInLoop()，从而造成死锁。
   
  {
    MutexLockGuard lock(mutex_);
    functors.swap(pendingFunctors_); // 直接swap出来，减少临界区 这也值得学习的地方
  }

  for (size_t i = 0; i < functors.size(); ++i) // 招待回调
  {
    functors[i]();
  }
  callingPendingFunctors_ = false;
}
```

这就完成了线程之间的事件通知。然后把事务交给它所属的线程进行执行的过程。


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






