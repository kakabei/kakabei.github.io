---
layout: post
title: muduo笔记  第八章  一些模块常有(void)n 的做法
date: 2018-01-05 12:21:12
categories: Linux muduo
tags:  系统编程 muduo笔记 技术阅读笔记 tcp network thread
excerpt: muduo TimerQueue 定时器
---




看了muduo的好几个模块出现类似于 (void) n的做法。
```c++
// update existing one with EPOLL_CTL_MOD/DEL
    int fd = channel->fd();
    (void)fd;		// 这个地方是什么意思？

```

```c++
std::vector<TimerQueue::Entry> TimerQueue::getExpired(Timestamp now)
{
  assert(timers_.size() == activeTimers_.size());
  std::vector<Entry> expired;
  Entry sentry(now, reinterpret_cast<Timer*>(UINTPTR_MAX));
  TimerList::iterator end = timers_.lower_bound(sentry);
  assert(end == timers_.end() || now < end->first);
  std::copy(timers_.begin(), end, back_inserter(expired));
  timers_.erase(timers_.begin(), end);

  for (std::vector<Entry>::iterator it = expired.begin();
      it != expired.end(); ++it)
  {
    ActiveTimer timer(it->second, it->second->sequence());
    size_t n = activeTimers_.erase(timer);
    assert(n == 1); (void)n; // 这里也出现了 对一个 整型 进行(void)
  }

  assert(timers_.size() == activeTimers_.size());
  return expired;
}

```

知乎上有人问题了muduo的作者。
[muduo库的很多返回值为空的函数的最后一行都有类似 (void) n的一句， 为什么要这样做？](https://www.zhihu.com/question/24311085)

这是提问者的回答：
原来作者是把gcc的编译严格等级设置为任何警告当成错误，而有的变量实际上是在debug版本中assert时有用的，release时这个变量就没用了， 此时编译器就会提醒有个变量没用的警告。这时就会被当做错误了。所以用(void)n 避免这个错误。 感觉陈硕的回应好冷。呵呵
 

---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






