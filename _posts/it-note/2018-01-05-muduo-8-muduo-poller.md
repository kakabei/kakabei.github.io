---
layout: post
title: muduo笔记 第八章  Poller 的理解
date: 2018-01-05 08:21:12
categories: muduo
tags:  系统编程 muduo 技术阅读笔记 
excerpt: muduo  Poller 的理解
---


Poller 可以看成是对 epoll和poll的方式的封装。是多路复用的基类。

muduo 的做法是  Polller 是一个基类。而 epoll 封装在EPollPoller中， poll 封装在PollPoller中， 而EPollPoller 和PollPoller都继承了Polller。

Polller 核心是：

```c++
typedef std::vector<struct pollfd> PollFdList;
typedef std::map<int, Channel*> ChannelMap;  // fd to Channel
PollFdList pollfds_;   //  不在基类中 而是在PollPoller 中 而EPollPoller 中 EventList 都存在各的类的中，
ChannelMap channels_;

```

ChannelMap 是对Channel的映射。用fd对应着一个Channel。从这里也说明了一个Channel对应关一个fd。

```c++
typedef std::vector<struct epoll_event> EventList; //事件列表

int epollfd_;
EventList events_;
```

所有的evntes_ 都是从Channel中来的


Polller 主要的方法：(有个三个是虚函数)

```c++
virtual Timestamp poll(int timeoutMs, ChannelList* activeChannels);
virtual void updateChannel(Channel* channel);
virtual void removeChannel(Channel* channel);

void fillActiveChannels(int numEvents,
                          ChannelList* activeChannels) const;
void update(int operation, Channel* channel);
```

updateChannel和removeChannel都是对上面两个数据结构的操作，poll函数是对::poll的封装。私有的fillActiveChannels函数负责把返回的活动时间添加到activeChannels（vector<Channel*>）这个结构中，返回给用户。Poller的职责也很简单，负责IO multiplexing，一个EventLoop有一个Poller，Poller的生命周期和EventLoop一样长。

Poller 的主要逻辑在poll中。它主要是等待事件，然后把事件放到活跃列表中。

poll分别封装了epoll_wait  和 poll 等待事件的方法。

```c++
// EPollPoller
Timestamp EPollPoller::poll(int timeoutMs, ChannelList* activeChannels)
{
  LOG_TRACE << "fd total count " << channels_.size();
  // 等待事件 epoll_wait 被触发的所有事件入在events_ 中
  int numEvents = ::epoll_wait(epollfd_,
                               &*events_.begin(),
                               static_cast<int>(events_.size()),
                               timeoutMs);
  int savedErrno = errno;
  Timestamp now(Timestamp::now());
  if (numEvents > 0)
  {
    LOG_TRACE << numEvents << " events happended";
    fillActiveChannels(numEvents, activeChannels); //找出所有活跃的事件fd
    if (implicit_cast<size_t>(numEvents) == events_.size())
    {
      events_.resize(events_.size()*2); // 扩展一倍
    }
  }
  else if (numEvents == 0)
  {
    LOG_TRACE << "nothing happended";
  }
  else
  {
    // error happens, log uncommon ones
    if (savedErrno != EINTR)
    {
      errno = savedErrno;
      LOG_SYSERR << "EPollPoller::poll()";
    }
  }
  return now;
}
```

```c++
// PollPoller
Timestamp PollPoller::poll(int timeoutMs, ChannelList* activeChannels)
{
  // XXX pollfds_ shouldn't change
  int numEvents = ::poll(&*pollfds_.begin(), pollfds_.size(), timeoutMs);
  int savedErrno = errno;
  Timestamp now(Timestamp::now());
  if (numEvents > 0)
  {
    LOG_TRACE << numEvents << " events happended";
    fillActiveChannels(numEvents, activeChannels);
  }
  else if (numEvents == 0)
  {
    LOG_TRACE << " nothing happended";
  }
  else
  {
    if (savedErrno != EINTR)
    {
      errno = savedErrno;
      LOG_SYSERR << "PollPoller::poll()";
    }
  }
  return now;
}
```

感觉 最不好理解的就是updateChannel这个方法了。两个继承类中updateChannel对indix_的定义不一样。
但大致上都是因 channel的信息变化了 对 ChannelMap 的更新。
还感觉这一个地方写的不太好。


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






