---
layout: post
title: muduo笔记  第八章 TcpConnection class 发送数据
date: 2018-01-05 19:22:12
categories: Linux muduo
tags:  系统编程 muduo笔记 技术阅读笔记 tcp network thread
excerpt: muduo TcpConnection class 发送数据
---

TcpConnection class 可谓 是 muduo 最 核心 也是 最 复杂 的 class，

TcpConnection 是 muduo 里唯一默认使用shared_ ptr来管理的class， 也是唯一继承enable_shared_from_this 的class，这源于其模糊的生命期。

注意TcpConnection表示的是“ 一次TCP连接”， 它是不可再生 的， 一旦连接断开，这个 TcpConnection 对象就没啥用了。 另外TcpConnection没有发起连接的功能，其构造函数的参数是已经建立好连接的 socket fd（无论是TcpServer被动接受还是TcpClient主动发起），因此 其初始状态是kConnecting。



TcpConnection 状态的变化。

![](/assets/muduo/8-muduo-tcpserver-close-connection.png) 

发送数据比接收数据更难，因为发送数据是主动的，接收读取数据是被动的。
由于muduo采用leveltrigger，因此我们只在需要时才关注writable事件，否则就会造成busyloop。


sendInLoop()会先尝试直接发送数据，如果一次发送完毕就不会启用WriteCallback；如果只发送了部分数据，则把剩余的数据放入outputBuffer_，并开始关注writable事件，以后在handlerWrite()中发送剩余的数据。如果当前outputBuffer_已经有待发送的数据，那么就不能先尝试发送了，因为这会造成数据乱序。

```c++
/// 处理写事件的回调。 同样， 没有while() 能一次写完吗？
/// ---
/// 这里，是在由于sendInLoop发送数据一次没有发送完，把余下的数据写入
/// outputBuffer_ 中，然再设置channel定事件，回调到这个来了。

void TcpConnection::handleWrite()
{
  loop_->assertInLoopThread();
  if (channel_->isWriting())
  {
    ssize_t n = sockets::write(channel_->fd(),
                               outputBuffer_.peek(),
                               outputBuffer_.readableBytes());
    if (n > 0)
    {
      outputBuffer_.retrieve(n);
      if (outputBuffer_.readableBytes() == 0) // 写完了
      {
         //如果写完了就关闭写事件， 没写完不关闭，则写事件会一直被触发
        channel_->disableWriting(); 
        if (writeCompleteCallback_)
        {
          loop_->queueInLoop(boost::bind(writeCompleteCallback_, shared_from_this()));
        }
        if (state_ == kDisconnecting)
        {
          shutdownInLoop(); // 正在关闭中
        }
      }
    }
    else
    {
      LOG_SYSERR << "TcpConnection::handleWrite";
      /// 另外如果这时连接正在关闭（L161）， 则调用shutdownInLoop()，继续执行关闭过程。
      /// 这里不需要处理错误，因为一旦发生错误，handleRead()会读到0字节，继而关闭连接。

      // if (state_ == kDisconnecting)
      // {
      //   shutdownInLoop();
      // }
    }
  }
  else
  {
    LOG_TRACE << "Connection fd = " << channel_->fd()
              << " is down, no more writing";
  }
}
```

当socket变得可写时，Channel会调用TcpConnection::handleWrite()，这里我们继续发送outputBuffer_中的数据。

一旦发送完毕，立刻停止观察writable事件（L160），避免busyloop。另外如果这时连接正在关闭（L161），则调用shutdownInLoop()，继续执行关闭过程。

这里不需要处理错误，因为一旦发生错误，handleRead()会读到0字节，继而关闭连接。

注意sendInLoop()和handleWrite()都只调用了一次write(2)而不会反复调用直至它返回EAGAIN，原因是如果第一次write(2)没有能够发送完全部数据的话，第二次调用write(2)几乎肯定会返回EAGAIN。

读者可以很容易用下面的Python代码来验证这一点。因此muduo决定节省一次系统调用，这么做不影响程序的正确性，却能降低延迟。


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.






