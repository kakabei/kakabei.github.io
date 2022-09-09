---
layout: post
title: muduo笔记  第八章 TimerQueue 定时器
date: 2018-01-05 10:21:12
categories: muduo
tags:  系统编程 muduo笔记 技术阅读笔记 tcp network thread
excerpt: muduo TimerQueue 定时器
---


muduo 对于时间的摆选择可看  [muduo笔记 第七章 定时器](http://blog.xyecho.com/muduo-7-muduo-timer/)

muduo 的 定时器功能由 三个class实现，`TimerId、Timer、TimerQueue`， 用户只能看到第一个class， 另外两个都是内部实现细节。

Timer中 有用到 Timestamp class  这个一个封装了微秒的时间戳类。	 

**其中now() 调用的是 gettimeofday() 这个函数不是系统调用，而是在用户态实现的。 没有上下文切换和陷入内核的开销。**

被真正用函数是EventLoop 的：

```c++
 TimerId runAt(const Timestamp& time, TimerCallback&& cb);
 TimerId runAfter(double delay, TimerCallback&& cb);
 TimerId runEvery(double interval, TimerCallback&& cb);
 void cancel(TimerId timerId);
```

这一些函数都是线程安全的。

### 定时器Timer

Timer 是一个定时器类。
主要成员有：

```c++
const TimerCallback callback_;   // 时间回调函数
Timestamp expiration_;           // 时间  （微秒）
const double interval_;          // 字面的理解 时间间隔
const bool repeat_;              //  如果true 就是把interval_加到expiration_ 
const int64_t sequence_;         //  定时器序号 这是什么场景用的？

```
定时器的回调函数是在被设置在Timer中的。

### 时间管理队列 TimerQueue 

应该可以看着是对Timer的管理类吧。最核心的地方应该就是对时间的管理 ，主要是排序，就是怎么能快速打到已经到期的Timer。也能高效的增加和删除Timer.

TimerQueue 排行是用stl set。 
key值是一个pair <Timestamp, Timer*> 。 这样解决了如果时间相等的情况下，第二个地址也不可能相等。
```c++
typedef std::pair<Timestamp, Timer*> Entry;
typedef std::set<Entry> TimerList;
TimerList timers_;  // Timer list sorted by expiration 按超时的时间顺序
```

TimerQueue 的主要方法 是 : 

```c++
TimerId addTimer(const TimerCallback& cb,
                   Timestamp when,
                   double interval);

```

这代码中有一个地方应该要注意到的是：
```c++
// 这个函数直接的返回了局部的 vector 这个做没有什么问题吗？
// 
// rvo realease版本优化 所以不需要返回引用或指针
// RVO:即release版本中会对代码优化 使得返回一个对象不许调用拷贝构造函数 所以可以返回一个对象 不用指针或者引用

std::vector<TimerQueue::Entry> TimerQueue::getExpired(Timestamp now)
{
  assert(timers_.size() == activeTimers_.size());
  std::vector<Entry> expired;
  Entry sentry(now, reinterpret_cast<Timer*>(UINTPTR_MAX));
  TimerList::iterator end = timers_.lower_bound(sentry);
  assert(end == timers_.end() || now < end->first);    // 在这里做比较， 比now大的所有Entry
  // 这里是从一个set拷贝到vector
  // 最新版本改为如下：
  // expired.assign(timers_.begin(), end);
  std::copy(timers_.begin(), end, back_inserter(expired));
  timers_.erase(timers_.begin(), end);

  for (std::vector<Entry>::iterator it = expired.begin();
      it != expired.end(); ++it)
  {
    ActiveTimer timer(it->second, it->second->sequence());
    size_t n = activeTimers_.erase(timer);
    assert(n == 1); (void)n; // 这里也出现了 对一个 整型 进行(void)?  防变量未设置错误
  }

  assert(timers_.size() == activeTimers_.size());
  return expired;
}
```

这个函数的作用是取出过期的Timer列表。 可以它的返回了一个函数内的局部vector。以后是有问题，但又觉得，一个还算比较流行的源开放项目，不太可能会出现这样的问题。查了一下，才知道，还是自己的知识有限啊。 这种方式叫vo realease版本优化。[rvo realease版本优化](http://blog.xyecho.com/muduo-8-muduo-rvo-realease/)

关于对 TimerList timers_的一些操作也变蛮不好理解的。

是 TimerQueue 回 调用户代码onTimer() 的时序图。

![](/assets/muduo/8-timer-queue.png) 


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






