---
layout: post
title: muduo笔记  第八章 TcpServer TcpConnection 断开连接
date: 2018-01-05 19:21:12
categories: muduo
tags:  系统编程 muduo 技术阅读笔记 
excerpt: muduo TcpServer TcpConnection 断开连接
---

muduo关闭连接的方式有两种：被动关闭（见此处）。即对方先关闭连接， 本地read( 2) 返回 0，触发关闭逻辑。
另一种是主动关闭。

![](/assets/muduo/8-muduo-tcpserver-close-connection.png) 

删除连接的方式比较复杂，是真的很很复杂啊。

TcpConnection类是用share_ptr来管理的。所以总结成最后一句话就是：**当share_ptr没有被引用，计数下降到0时TcpConnection对象就会被释放。连接就会被断开。**

TcpServer::removeConnection()把conn从ConnectionMap中移除。这时TcpConnection已经是命悬一线：如果用户不持有TcpConnectionPtr的话，conn的引用计数已降到1。注意这里一定要用EventLoop::queueInLoop()，否则就会出现此处讲的对象生命期管理问题。

### 从头说起。 
---
TcpConnection在处理读事件时，如果读到的数据为0，则说明对端已经发起关闭连接。

```c++
/// 处理读出的数据 回调messageCallback_
/// 这个地方一次能读完吗？为什么没有while()?? -- 2018-09-17 // epoll模式是level trigger 如果fd还有可读的，就会继续触发。
void TcpConnection::handleRead(Timestamp receiveTime)
{
  loop_->assertInLoopThread();
  int savedErrno = 0;
  ssize_t n = inputBuffer_.readFd(channel_->fd(), &savedErrno);
  if (n > 0)
  {
    messageCallback_(shared_from_this(), &inputBuffer_, receiveTime);
  }
  else if (n == 0) // 链接被动关闭
  {
    handleClose();
  }
  else
  {
    errno = savedErrno;
    LOG_SYSERR << "TcpConnection::handleRead";
    handleError();
  }
}
```

则在handleClose()中处理。 注释中的说明为什么没有while() 其实不一定一次能读完， 只是epoll是level trigger 还会继续触发。

```c++
/// 两种情况 会关闭链接 1) 读到0时 2)server 主动关闭
/// 关闭时的处理 要处理理回调函数  closeCallback_
/// 这里并不没有关闭fd ?
void TcpConnection::handleClose()
{
  loop_->assertInLoopThread();
  LOG_TRACE << "fd = " << channel_->fd() << " state = " << stateToString();
  assert(state_ == kConnected || state_ == kDisconnecting);
  // we don't close fd, leave it to dtor, so we can find leaks easily.
  setState(kDisconnected);
  channel_->disableAll();

  TcpConnectionPtr guardThis(shared_from_this());  // 一个指向自己的智能指针share
  connectionCallback_(guardThis);       // 回调回TCPserver 做处理
  // must be the last line
  closeCallback_(guardThis); // 关闭时的回调 TcpServer::removeConnection 要在connections_删除掉这个链接
}
```

这里并没有看到关闭socket的fd。是因为TcpConnection是交给share_ptr管理的。 当share_ptr没有被引用，计数下降到0时TcpConnection对象就会被释放。连接就会被断开。

设置状态为 kDisconnected。 设置channel的状态。
这个关键一步是  closeCallback_(guardThis);  关闭时的回调 TcpServer::removeConnection 要在connections_删除掉这个链接

在void TcpServer::newConnection 就已经设置了closeCallback_的回调。

```c++
conn->setCloseCallback(
      boost::bind(&TcpServer::removeConnection, this, _1)); // FIXME: unsafe
```
所以执行是TcpServer::removeConnection()

```c++
/// 销毁链接
void TcpServer::removeConnection(const TcpConnectionPtr& conn)
{
  // FIXME: unsafe 不安全？ 
  loop_->runInLoop(boost::bind(&TcpServer::removeConnectionInLoop, this, conn));
}

void TcpServer::removeConnectionInLoop(const TcpConnectionPtr& conn)
{
  loop_->assertInLoopThread();
  LOG_INFO << "TcpServer::removeConnectionInLoop [" << name_
           << "] - connection " << conn->name();
  size_t n = connections_.erase(conn->name());  
  (void)n;
  assert(n == 1);
  EventLoop* ioLoop = conn->getLoop();
  ioLoop->queueInLoop(
      boost::bind(&TcpConnection::connectDestroyed, conn));
}
```

这里它擦去了connections_中的这个连接。 size_t n = connections_.erase(conn->name());  
然后， 调用TcpConnection::connectDestroyed()

```c++
/// connectDestroyed() 是TcpConnection析构前最后调用的一个成员函数，它通知用户连接已断开。
void TcpConnection::connectDestroyed()
{
  loop_->assertInLoopThread();
  if (state_ == kConnected)
  {
    setState(kDisconnected);
    channel_->disableAll();

    connectionCallback_(shared_from_this()); // 只是打印了日志
  }
  channel_->remove();
}
```

channel_->remove(); 把对应的channel 删除掉。

```c++
void Channel::remove()
{
  assert(isNoneEvent());
  addedToLoop_ = false;
  loop_->removeChannel(this);
}
```

当channel被删除掉后，TcpConnection对象就没有引用了。share_ptr的计数就到0.对象被析构掉了。

要看清楚， TcpConnection被多少个地方引用。把这些地方都删除掉的话。就会自动把对像析构掉。

TcpConnection对象被析构后，对象socket也会被析构掉，在socket的析构函数中 调用了close()

```c++
Socket::~Socket()
{
  sockets::close(sockfd_);
}
```

到这里连接断开了。

forceClose()是主动关闭连接。


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






