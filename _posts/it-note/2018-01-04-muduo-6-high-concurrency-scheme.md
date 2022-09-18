---
layout: post
title: muduo笔记 第六章 常见的并发网络服务器程序设计方案
date: 2018-01-04 20:08:12
categories: muduo
tags:  技术阅读笔记 系统编程 muduo tcp network thread
excerpt: 常见的并发网络服务器程序设计方案
---


#### 常见的并发网络服务器程序设计方案：

![](/assets/muduo/high-concurrency-scheme1.png) 

#### 方案0

一次只能服务一个链接。现在的网络服务器基本不会这么做，所以这种方案基本是废的。

```python
#!/usr/bin/python

import socket

def handle(client_socket, client_address):
    while True:
        data = client_socket.recv(4096)
        if data:
            sent = client_socket.send(data)    # sendall?
        else:
            print "disconnect", client_address
            client_socket.close()
            break

if __name__ == "__main__":
    listen_address = ("0.0.0.0", 2007)
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind(listen_address)
    server_socket.listen(5)

    while True:
        (client_socket, client_address) = server_socket.accept()
        print "got connection from", client_address
        handle(client_socket, client_address)
```

#### 方案1

这个方案就是一个进程服务一个客户端，　称之为child-per-client或fork()-per-client，另外也俗称process-per-connection。

这种方案适合并发连接数不大的情况。至今仍有一些网络服务程序用这种方式实现，比如PostgreSQL和Perforce的服务端。

这种方案适合"计算响应的工作量远大于fork()的开销"这种情况，比如数据库服务器。这种方案适合长连接，但不太适合短连接，因为fork()开销大于求解Sudoku的用时。

```python 
#!/usr/bin/python
from SocketServer import BaseRequestHandler, TCPServer
from SocketServer import ForkingTCPServer, ThreadingTCPServer

class EchoHandler(BaseRequestHandler):
    def handle(self):
        print "got connection from", self.client_address
        while True:
            data = self.request.recv(4096)
            if data:
                sent = self.request.send(data)    # sendall?
            else:
                print "disconnect", self.client_address
                self.request.close()
                break

if __name__ == "__main__":
    listen_address = ("0.0.0.0", 2007)
    server = ForkingTCPServer(listen_address, EchoHandler)
    server.serve_forever()
```

#### 方案2
 
一个线程对应一个连接。一个客户端链接上来，就会开一个线程为它服务。

Java网络服务多采用这种方案。它的初始化开销比方案1要小很多，但与求解Sudoku的用时差不多，仍然不适合短连接服务。

这种方案的伸缩性受到线程数的限制，一两百个还行，几千个的话对操作系统的scheduler恐怕是个不小的负担。

#### 方案3 方案4

方案3是对方案1的优化。

方案4是对方案2的优化。

这两种方案是apache httpd长期使用的方案。

#### 方案5 

基本的单线程Reactor方案。 是IO复用，非阻塞的了。用注册回调可以实现网络和业务的分离。
\
这种方案的优点是由网络库搞定数据收发，程序只关心业务逻辑；缺点在前面已经谈了：适合IO密集的应用，不太适合CPU密集的应用，因为较难发挥多核的威力。

另外，与方案2相比，方案5处理网络消息的延迟可能要略大一些，因为方案2直接一次read(2)系统调用就能拿到请求数据，而方案5要先poll(2)再read(2)，多了一次系统调用。

#### 方案6 

是一个过渡方案。

收到请求之后，不在Reactor线程计算，而是创建一个新线程去计算，以充分利用多核CPU。这是非常初级的多线程应用，因为它为每个请求（而不是每个连接）创建了一个新线程。

这个开销可以用线程池来避免。

#### 方案7

为每个连接创建一个计算线程，每个连接上的请求固定发给同一个线程去算，先到先得。这也是一个过渡方案，因为并发连接数受限于线程数目，这个方案或许还不如直接使用阻塞IO的thread-per-connection方案2。

##### 方案8

为了弥补方案6中为每个请求创建线程的缺陷，我们使用固定大小线程池，程序结构如图6-12所示。全部的IO工作都在一个Reactor线程完成，而计算任务交给thread pool。如果计算任务彼此独立，而且IO的压力不大，那么这种方案是非常适用的。

![](/assets/muduo/high-concurrency-scheme2.png) 


##### 方案9 

这是muduo内置的多线程方案，也是Netty内置的多线程方案。这种方案的特点是one loop per thread，有一个main Reactor负责accept(2)连接，然后把连接挂在某个sub Reactor中（muduo采用round-robin的方式来选择sub Reactor），这样该连接的所有操作都在那个sub Reactor所处的线程中完成。多个连接可能被分派到多个线程中，以充分利用CPU。

##### 方案10 　

这是Nginx的内置方案。如果连接之间无交互，这种方案也是很好的选择。工作进程之间相互独立，可以热升级。

##### 方案11 　

把方案8和方案9混合，既使用多个Reactor来处理IO，又使用线程池来处理计算。这种方案适合既有突发IO（利用多线程处理多个连接上的IO），又有突发计算的应用（利用线程池把一个连接上的计算任务分配给多个线程去做），见图6-14。

![](/assets/muduo/high-concurrency-scheme3.png) 

一个程序到底是使用一个event loop还是使用多个event loops呢？

ZeroMQ的手册给出的建议是 ，按照每千兆比特每秒的吞吐量配一个event loop的比例来设置event loop的数目，即muduo::TcpServer::setThreadNum()的参数。

依据这条经验规则，在编写运行于千兆以太网上的网络程序时，用一个event loop就足以应付网络IO。

如果程序本身没有多少计算量，而主要瓶颈在网络带宽，那么可以按这条规则来办，只用一个event loop。

另一方面，如果程序的IO带宽较小，计算量较大，而且对延迟不敏感，那么可以把计算放到thread pool中，也可以只用一个event loop。


ZeroMQ的手册给出的建议 : [http://zeromq.org/area:faq#toc3](http://zeromq.org/area:faq#toc3)

---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.