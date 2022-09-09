---
layout: post
title: muduo笔记  第八章 Channel 的理解
date: 2018-01-05 09:21:12
categories: muduo
tags:  系统编程 muduo笔记 技术阅读笔记 tcp network thread
excerpt: muduo  Channel 的理解
---

Channel 主要是负责对I/O事件的分发。 一个Channel对象对应一个fd。

对Channle的误解可能是一开始时就想成了管道，渠道带缓存之类的。但其实和这个一点毛关系也没有。

Channel 的 核心如下：

```c++
EventLoop* loop_;
const int fd_;
int events_;
int revents_;
int index_;

ReadEventCallback readCallback_;
EventCallback writeCallback_;
EventCallback errorCallback_;
EventCallback closeCallback_;
```

EventCallback 会在handleEvent这个成员函数中根据不同的事件被调用。

index_ 比较不好理解一点，在Poll和Epoll两个方式下index_的意义还不一样。
 - 在PollPoller类中pollfds_数组的下标。
 - 在EPollPoller类中 可三种不同的状态 kNew  kAdded kDeleted。

在events_和revents_明显对应了struct pollfd结构中的成员。其实poll和 epoll的事件编号应该一样的。muduo 在EPollPoller中
做了检测：

```
BOOST_STATIC_ASSERT(EPOLLIN == POLLIN);
BOOST_STATIC_ASSERT(EPOLLPRI == POLLPRI);
BOOST_STATIC_ASSERT(EPOLLOUT == POLLOUT);
BOOST_STATIC_ASSERT(EPOLLRDHUP == POLLRDHUP);
BOOST_STATIC_ASSERT(EPOLLERR == POLLERR);
BOOST_STATIC_ASSERT(EPOLLHUP == POLLHUP);

```

需要注意的是，Channel并不拥有该fd,它不会在析构函数中去关闭这个fd（fd是由Socket类的析构函数中关闭，即RAII的方法），Channel的生命周期由其owner负责。
fd 在对象初始化时给出。但在析构时并不去关闭它，而是由owner自己负责。

**后续要再看看加强理解**

---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






