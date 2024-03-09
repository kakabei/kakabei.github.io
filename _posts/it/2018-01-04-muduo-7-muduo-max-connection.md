---
layout: post
title: muduo笔记 第七章 限制服务器的最大并发连接数
date: 2018-01-04 20:19:12
categories: muduo
tags:  技术学习笔记 系统编程 muduo 
excerpt: 限制服务器的最大并发连接数
--- 


一方面， 我们不希望服务程序超载。

另一方面，更因为filedescriptor是 稀缺资源， 如果出现 filedescriptor耗尽，很棘手，跟"malloc() 失败/new抛出std::bad_alloc"差不多同样棘。

当accept(2) 返回EMFILE该如何应对？

这意味着本进程的文件描述符已经达到上限， 无法为新连接创建socket文件描述符。

但是，既然没有socket文件描述符来表示这个连接，我们就无法close(2) 它。

程序继续运行，回到L11再一次调用epoll_ wait。这时候epoll_wait会立刻返回，因为新连接还等待处理，listening fd还是可读的。这样程序 立刻就陷入了busy loop， CPU占用率接近100%。 

这既影响同一event loop上的连接， 也影响同一机器上的其他服务。

#### 解决方法

准备一个空闲的文件描述符。 遇到这种情况，先关闭这个空闲文件， 获得一个文件描述符的名额；

再accept(2) 拿到新socket连接的描述符；随后立刻close(2) 它，这样就优雅地断开了客户端连接；

最后重新打开一个空闲文件， 把"坑"占住，以备再次出现这种情况时使用。

其实有另外一种比较简单的办法： 

file descriptor是hard limit，我们可以自己设一个 稍低 一点 的 soft limit， 

如果超过soft limit 就主动关闭新连接， 这样就可避免触及"file descriptor 耗尽" 这种边界条件。

```c
if (errno == EMFILE)
    {
      ::close(idleFd_);
      idleFd_ = ::accept(acceptSocket_.fd(), NULL, NULL);
      ::close(idleFd_);
      idleFd_ = ::open("/dev/null", O_RDONLY | O_CLOEXEC);
    }
```

#### [注]
1、可能我们在实践当中，中不太可能改动每个服务器的，特别当服务器越来越多时，根本不靠谱。所以在框架中解决这个问题是比较好的选择。


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.




