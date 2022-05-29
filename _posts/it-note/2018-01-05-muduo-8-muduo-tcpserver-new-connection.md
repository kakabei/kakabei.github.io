---
layout: post
title: muduo笔记  第八章 TcpServer 接受新连接
date: 2018-01-05 18:21:12
categories: Linux muduo
tags:  系统编程 muduo笔记 技术阅读笔记 tcp network thread
excerpt: muduo TcpServer 接受新连接
---

TcpServer新建连接的相关函数调用顺序见图

![](/assets/muduo/8-muduo-tcpserver-new-connection.png) 


TcpServer class的功能是管理accept(2) 获得的TcpConnection。 TcpServer是供用户直接使用的， 生命期由用户控制。

用户只需要设置好callback，再调用start() 即可。

---

TcpServer内部使用Acceptor来获得新连接的fd。 它保存用户提供的ConnectionCallback和MessageCallback，在新建TcpConnection的时候会原样传给后者。

TcpServer持有目前存活的TcpConnection的shared_ ptr（定义为TcpConnectionPtr），因为TcpConnection对象的生命期是模糊的，用户也可以持有TcpConnectionPtr。

每个TcpConnection对象有一个名字，这个名字是由其所属的TcpServer在创建TcpConnection对象时生成，名字是ConnectionMap的key。

在新连接到达时，Acceptor会回调newConnection()，**[newConnection()是TcpServer类的成员函数，被调设置到Acceptor的回调中]** 

后者会创建TcpConnection对象conn，把它加入ConnectionMap，设置好callback，再调用conn->connectEstablished()，其中会回调用户提供的ConnectionCallback。

---

TcpConnection 类的一部分成员：

```c++
class TcpConnection : boost::noncopyable,
                      public boost::enable_shared_from_this<TcpConnection>
{
private:
  enum StateE { kDisconnected, kConnecting, kConnected, kDisconnecting };
  void handleRead(Timestamp receiveTime);
  void handleWrite();
  void handleClose();
  void handleError();
  // void sendInLoop(string&& message);
  void sendInLoop(const StringPiece& message);
  void sendInLoop(const void* message, size_t len);
  void shutdownInLoop();
  // void shutdownAndForceCloseInLoop(double seconds);
  void forceCloseInLoop();
  void setState(StateE s) { state_ = s; }
  const char* stateToString() const;
  void startReadInLoop();
  void stopReadInLoop();

  EventLoop* loop_;
  const string name_;
  StateE state_;  // FIXME: use atomic variable
  bool reading_;
  // we don't expose those classes to client.
  boost::scoped_ptr<Socket> socket_;
  boost::scoped_ptr<Channel> channel_;
  const InetAddress localAddr_;
  const InetAddress peerAddr_;
  
  // 一些回调函数
  ConnectionCallback connectionCallback_;        // 这个回调是干吗的呢？  就打印日志？
  MessageCallback messageCallback_;              // 消息处理的回调

  /// 发送缓存区的高水位回调 highWaterMarkCallback_
  /// 发送缓存区的低水位回调 writeCompleteCallback_
  WriteCompleteCallback writeCompleteCallback_;  // 写数据完成的回调 发送缓存区的低水位回调
  HighWaterMarkCallback highWaterMarkCallback_;  // ??? 感觉这个就回调给上一层处理，如一个提示之类的
  CloseCallback closeCallback_;                  // 关闭链接的回调
  
  size_t highWaterMark_;
  Buffer inputBuffer_;   // 从fd读出的数据缓存
  Buffer outputBuffer_; // FIXME: use list<Buffer> as output buffer.  // 从要发送出去的缓存
  boost::any context_;
  // FIXME: creationTime_, lastReceiveTime_
  //        bytesReceived_, bytesSent_
};

typedef boost::shared_ptr<TcpConnection> TcpConnectionPtr;

```

#### 从头说起一个新链接的创建。

创建出TcpServer 的对象。TcpServer 在构造函数中就 new 出Acceptor() 。又把 &TcpServer::newConnection 设置给 Acceptor的回调了。

```c++
// 把 newConnection()设置给 acceptor
  acceptor_->setNewConnectionCallback(
      boost::bind(&TcpServer::newConnection, this, _1, _2));
```

看Acceptor发生了什么事。[Acceptor class](http://blog.xyecho.com/muduo-8-muduo-acceptor-class/)

在Acceptor的构造函数中 调用了sockets::createNonblockingOrDie() 这里面就是调用::socket() 这时socket被成功创建了。

```c++
Acceptor::Acceptor(EventLoop* loop, const InetAddress& listenAddr, bool reuseport)
  : loop_(loop),
    acceptSocket_(sockets::createNonblockingOrDie(listenAddr.family())),
    acceptChannel_(loop, acceptSocket_.fd()),
    listenning_(false),
    idleFd_(::open("/dev/null", O_RDONLY | O_CLOEXEC)) //打开一个做了空闲的fd
{
  assert(idleFd_ >= 0);
  acceptSocket_.setReuseAddr(true);
  acceptSocket_.setReusePort(reuseport);
  acceptSocket_.bindAddress(listenAddr);
  acceptChannel_.setReadCallback(
      boost::bind(&Acceptor::handleRead, this));
}
```

#### 当TcpServer 的对象调用start()时。

```c++
// 服务开始的地方 执行listen()
void TcpServer::start()
{
  if (started_.getAndSet(1) == 0) // 什么意思？
  {
    threadPool_->start(threadInitCallback_); //把回调函数传入

    assert(!acceptor_->listenning());
    loop_->runInLoop(
        boost::bind(&Acceptor::listen, get_pointer(acceptor_)));   
  }
}
```

这个时候TCPServer开始了listen()了。 这个时候TCPSever应该算是启来了。就等着accept()新的连接了。

#### 下面要知道的是muduo是如何accept()的呢？

在Acceptor的构造函数中 acceptChannel_(loop, acceptSocket_.fd())。 已经把 socket的fd交给了channel了。同时也设置了

```c++
acceptChannel_.setReadCallback(
      boost::bind(&Acceptor::handleRead, this));
```

当acceptSocket_.fd()了读事件时，就会回调到 Acceptor::handleRead()

```c++
// 有了读事件，就回调函数 创建新的链接
void Acceptor::handleRead()
{
  loop_->assertInLoopThread();
  InetAddress peerAddr;
  //FIXME loop until no more
  int connfd = acceptSocket_.accept(&peerAddr);
  if (connfd >= 0)
  {
    // string hostport = peerAddr.toIpPort();
    // LOG_TRACE << "Accepts of " << hostport;
    if (newConnectionCallback_)
    {
      newConnectionCallback_(connfd, peerAddr); // 回调
    }
    else
    {
      sockets::close(connfd);
    }
  }
  else
  {
    LOG_SYSERR << "in Acceptor::handleRead";
    // Read the section named "The special problem of 
    // accept()ing when you can't" in libev's doc.
    // By Marc Lehmann, author of libev.

    // 这样做，只是为了优雅的断开客户端。但服务端的fd耗尽时，客户端的链接还是在的。
    //
    // 准备一个空闲的文件描述符。遇到这种情况，先关闭这个空闲文件，获得一个文件描述符的名额；
    // 再accept(2)拿到新socket连接的描述符；随后立刻close(2)它，这样就优雅地断开了客户端连接；
    // 最后重新打开一个空闲文件，把"坑"占住，以备再次出现这种情况时使用。

    if (errno == EMFILE)  // EMFILE
    {
      ::close(idleFd_);
      idleFd_ = ::accept(acceptSocket_.fd(), NULL, NULL);
      ::close(idleFd_);
      idleFd_ = ::open("/dev/null", O_RDONLY | O_CLOEXEC);
    }
  }
}
```

handleRead 回调 newConnectionCallback_() 就是TcpServer类中的TcpServer::newConnection().
后面是关于连接数过多的处理方式，可以看 [Acceptor class](http://blog.xyecho.com/muduo-8-muduo-acceptor-class/)

而TcpServer::newConnection()对新连接的处理。

```c++
/// 在Acceptor中，有读事件被触发时 在Acceptor::handleRead()会回调这个函数创建出一个新链接
void TcpServer::newConnection(int sockfd, const InetAddress& peerAddr)
{
  loop_->assertInLoopThread();
  EventLoop* ioLoop = threadPool_->getNextLoop(); // 从线程池中拿出一个 EventLoop
  char buf[64];
  snprintf(buf, sizeof buf, "-%s#%d", ipPort_.c_str(), nextConnId_);
  ++nextConnId_;
  string connName = name_ + buf;

  LOG_INFO << "TcpServer::newConnection [" << name_
           << "] - new connection [" << connName
           << "] from " << peerAddr.toIpPort();
  InetAddress localAddr(sockets::getLocalAddr(sockfd));
  // FIXME poll with zero timeout to double confirm the new connection
  // FIXME use make_shared if necessary
  TcpConnectionPtr conn(new TcpConnection(ioLoop,
                                          connName,
                                          sockfd,
                                          localAddr,
                                          peerAddr));
  connections_[connName] = conn;
  conn->setConnectionCallback(connectionCallback_);
  conn->setMessageCallback(messageCallback_);       // 处理链接发过了的数据函数
  conn->setWriteCompleteCallback(writeCompleteCallback_);
  conn->setCloseCallback(
      boost::bind(&TcpServer::removeConnection, this, _1)); // FIXME: unsafe
  ioLoop->runInLoop(boost::bind(&TcpConnection::connectEstablished, conn));
}
```

new出一个新TcpConnection 对象 然后 通过share_ptr的方式放入 ConnectionMap connections_ 中

```c++
 typedef std::map<string, TcpConnectionPtr> ConnectionMap;   // 链接管理map  <名字，TcpConnectionPtr> share指针
 ConnectionMap connections_;  // 链接管理map 
```

然后设置 conn的各种回调函数。

最后执行TcpConnection::connectEstablished 把 TcpConnection 绑定到 channel中 hannel_->tie(shared_from_this());

```c++
void TcpConnection::connectEstablished()
{
  loop_->assertInLoopThread();
  assert(state_ == kConnecting);
  setState(kConnected);
  channel_->tie(shared_from_this());
  channel_->enableReading();

  connectionCallback_(shared_from_this());
}
```

到这里，一个新连接的接受就完成了。

强制关闭和延迟关闭

```c++
/// 强制关闭
void TcpConnection::forceClose()
{
  // FIXME: use compare and swap
  if (state_ == kConnected || state_ == kDisconnecting)
  {
    setState(kDisconnecting);
    loop_->queueInLoop(boost::bind(&TcpConnection::forceCloseInLoop, shared_from_this()));
  }
}

/// 延迟强制关闭掉 延迟多少秒。 用的是定时器runAfter
void TcpConnection::forceCloseWithDelay(double seconds)
{
  if (state_ == kConnected || state_ == kDisconnecting)
  {
    setState(kDisconnecting);
    loop_->runAfter(
        seconds,
        makeWeakCallback(shared_from_this(),
                         &TcpConnection::forceClose));  // not forceCloseInLoop to avoid race condition
  }
}

/// 在自己所属的EventLoop关闭 链接
void TcpConnection::forceCloseInLoop()
{
  loop_->assertInLoopThread();
  if (state_ == kConnected || state_ == kDisconnecting)
  {
    // as if we received 0 byte in handleRead();
    handleClose();
  }
}
```

最后也是调用到handleClose()。后面的逻辑就一样了。

TcpConnection::shutdown() 只是关闭了定的动作。用的是Socket的 shutdown()。

```c++
/// shutdown和forceClose区别 ?
void TcpConnection::shutdown()
{
  // FIXME: use compare and swap
  if (state_ == kConnected)
  {
    setState(kDisconnecting);
    // FIXME: shared_from_this()?  
    //这里没有用到share_from_this ? 像forceClose 都是用到share_from_this的
    loop_->runInLoop(boost::bind(&TcpConnection::shutdownInLoop, this));
  }
}

/// 关闭链接， 这里只关闭写的动作 用的是  ::shutdown()
void TcpConnection::shutdownInLoop()
{
  loop_->assertInLoopThread();
  if (!channel_->isWriting()) // 如果当前没有发送数据
  {
    // we are not writing
    socket_->shutdownWrite();
  }
}

```


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






