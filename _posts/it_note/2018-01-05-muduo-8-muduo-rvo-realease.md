---
layout: post
title: muduo笔记 第八章 rvo realease版本优化
date: 2018-01-05 11:21:12
categories: Linux muduo
tags:  系统编程 muduo笔记 技术阅读笔记 tcp network thread
excerpt: muduo rvo realease版本优化
---



看TimerQueue  类时 看到一个函数如下：
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
  assert(end == timers_.end() || now < end->first);   //在这里做比较， 比now大的所有Entry
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
    assert(n == 1); (void)n; // 这里也出现了 对一个 整型 进行(void)? 
  }

  assert(timers_.size() == activeTimers_.size());
  return expired;
}

```

就如我第一次的注释，我很奇怪为什么直接返回了一个函数里的对象
后来才知道这叫RVO优化
什么时候应当依靠返回值优化（RVO）？
知乎上：
[什么时候应当依靠返回值优化（RVO）？]https://www.zhihu.com/question/27000013

---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






