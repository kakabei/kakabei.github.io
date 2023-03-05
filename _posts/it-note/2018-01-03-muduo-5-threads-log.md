---
layout: post
title: muduo笔记 第五章 高效的多线程日志
date: 2018-01-03 20:08:12
categories: muduo
tags:  技术学习笔记 系统编程 muduo
excerpt: 多线程日志
---

**"日志（logging）"有两个意思:**

1.诊断日志（diagnostic log） 　 即log4j、logback、slf4j、glog、g2log、log4cxx、log4cpp、log4cplus、Pantheios、ezlogger等常用日志库提供的日志功能。 

2.交易日志（transaction log） 　即数据库的write-ahead log 1 、文件系统的journaling 2 等，用于记录状态变更，通过回放日志可以逐步恢复每一次修改之后的状态。

本章的"日志"是前一个意思，即文本的、供人阅读的日志，通常用于故障诊断和追踪（trace） ，也可用于性能分析。

日志通常是分布式系统中事故调查时的唯一线索，用来追寻蛛丝马迹，查出元凶。


在服务端编程中，日志是必不可少的，在生产环境中应该做到"Log Everything All The Time" 。对于关键进程，日志通常要记录: 

- 收到的每条内部消息的id（还可以包括关键字段、长度、hash等）； 
- 收到的每条外部消息的全文； 
- 发出的每条消息的全文，每条消息都有全局唯一的id；
- 关键内部状态的变更，等等。


#### C++日志库的前端大体上有两种API风格：

C/Java的`printf(fmt, ...)`风格，例如 `log_info("Received %d bytes from %s", len, getClientName().c_str());`

C++的`stream <<` 风格，例如 `LOG_INFO << "Received " << len << " bytes from " << getClientName();`

muduo日志库是C++ stream风格的另一个好处是当输出的日志级别高于语句的日志级别时，打印日志是个空操作, 运行时开销接近零。
比方说当日志级别为WARNING时，`LOG_INFO <<`是空操作，这个语句根本不会调用`std::string getClientName()`函数，减小了开销。而printf风格不易做到这一点。 

可以看这篇文章 [Logging In C++](http://www.drdobbs.com/cpp/logging-in-c/201804215)

#### 性能需求

编写Linux服务端程序的时候，我们需要一个高效的日志库。只有日志库足够高效，程序员才敢在代码中输出足够多的诊断信息，减小运维难度，提升效率。

高效性体现在几方面： 

- 每秒写几千上万条日志的时候没有明显的性能损失。
- 能应对一个进程产生大量日志数据的场景，例如1GB/min。 
- 不阻塞正常的执行流程。 
- 在多线程程序中，不造成争用（contention）。

这里列举一些具体的性能指标，考虑往普通7200rpm SATA硬盘写日志文件的情况： 

磁盘带宽约是110MB/s，日志库应该能瞬时写满这个带宽（不必持续太久）。 

假如每条日志消息的平均长度是110字节，这意味着1秒要写100万条日志。

关于日志之前写过另一篇。可以看[日志模块](http://blog.xyecho.com/model-log/).


---
 \--- 《Linux多线程服务端编程：使用muduo C++ 网络库》 陈硕. 电子工业出版社. Kindle 版本.











