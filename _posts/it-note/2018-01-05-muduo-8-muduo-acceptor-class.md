---
layout: post
title: muduo笔记  第八章 Acceptor class
date: 2018-01-05 16:21:12
categories: muduo
tags:  系统编程 muduo 技术阅读笔记 
excerpt: muduo Acceptor class
---

Acceptor  class 是一个链接接收管理器。是一个内部使用类，供TcpServer  使用。

比较简单。

Acceptor的数据成员包括Socket、Channel等。 其中Socket是 一个RAIIhandle， 封装了socket文件描述符的生命期。 

Acceptor的socket是listening socket， 即 server socket。 Channel用于观察此socket上的readable事件， 并回调 Acceptor::handleRead()，后者会调用 accept(2) 来接受新连接，并回调用户callback。


在构造时，acceptChannel_(loop, acceptSocket_.fd()),
把 fd和loop通过Channel 绑定了。 
所以， 当有链接过来时，就可以处理事件了。事件的回调如下： 

```c++
// 设置新连接的回调函数，很重要一个接口
  void setNewConnectionCallback(const NewConnectionCallback& cb)
  { newConnectionCallback_ = cb; } 
```

有一个处理很好的地方是 当链接达到最大数时的处理方式：

```c++
    LOG_SYSERR << "in Acceptor::handleRead";
    // Read the section named "The special problem of
    // accept()ing when you can't" in libev's doc.
    // By Marc Lehmann, author of libev.

    // 这样做，只是为了优雅的断开客户端。但服务端的fd耗尽时，客户端的链接还是在的。
    //
    // 准备一个空闲的文件描述符。遇到这种情况，先关闭这个空闲文件，获得一个文件描述符的名额；
    // 再accept(2)拿到新socket连接的描述符；随后立刻close(2)它，这样就优雅地断开了客户端连接；
    // 最后重新打开一个空闲文件，把“坑”占住，以备再次出现这种情况时使用。

    if (errno == EMFILE)  // EMFILE
    {
      ::close(idleFd_);
      idleFd_ = ::accept(acceptSocket_.fd(), NULL, NULL);
      ::close(idleFd_);
      idleFd_ = ::open("/dev/null", O_RDONLY | O_CLOEXEC);
    }
```

对于Socket的封装后面再写了。


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






