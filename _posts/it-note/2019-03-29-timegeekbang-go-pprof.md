---
layout: post
title: 《极客时间》go语言核心36讲 程序性能分析基本
date: 2019-03-25 09:21:12
categories:  go 
tags:   go 极客时间笔记
excerpt: 条件变量是和互斥锁一起使用的。
---


Go 语言为程序开发者们提供了丰富的性能分析 API和标准工具。这些主要存在于： 

> 1 runtime/pprof
> 
> 2 net/http/pprof
> 
> 3 runtime/trace


在 Go 语言中，用于分析程序性能的概要文件有三种，分别是CPU 概要文件（CPU Profile）、内存概要文件（Mem Profile）和阻塞概要文件（Block Profile）。


对于 CPU 概要文件来说，其中的每一段独立的概要信息都记录着，在进行某一次采样的那个时刻，CPU 上正在执行的Go 代码。

而对于内存概要文件，其中的每一段概要信息都记载着，在某个采样时刻，正在执行的 Go 代码以及堆内存的使用情况，这里包含已分配和已释放的字节数量和对象数量。至于阻塞概要文件，其中的每一段概要信息，都代表着 Go 程序中的一个 goroutine 阻塞事件。

这时就可以显现出go tool pprof这个工具的作用了。我们可以通过它进入一个基于命令行的交互式界面，并对指定的概要文件进行查阅。概要文件是protoclo buffer 方式存储的。


**怎样设定内存概要信息的采样频率？**

只要为`runtime.MemProfileRate`变量赋值即可。

这个变量的含义是，平均每分配多少个字节，就对堆内存的使用情况进行一次采样。如果把该变量的值设为0，那么，Go 语言运行时系统就会完全停止对内存概要信息的采样。该变量的缺省值是512 KB，也就是512千字节。

越早设置越好，在main 开始时就设置。之后，需要调用`runtime/pprof`包中的`WriteHeapProfile`函数。该函数会把收集好的内存概要信息，写到我们指定的写入器中。

注意，我们通过`WriteHeapProfile`函数得到的内存概要信息并不是实时的，它是一个快照，是在最近一次的内存垃圾收集工作完成时产生的。如果你想要实时的信息，那么可以调用`runtime.ReadMemStats`函数。不过要特别注意，该函数会引起 Go 语言调度器的短暂停顿。


**怎样获取到阻塞概要信息？**

我们调用runtime包中的`SetBlockProfileRate`函数，即可对阻塞概要信息的采样频率进行设定。该函数有一个名叫rate的参数，它是int类型的。

这个参数的含义是，只要发现一个阻塞事件的持续时间达到了多少个纳秒，就可以对其进行采样。如果这个参数的值小于或等于0，那么就意味着 Go 语言运行时系统将会完全停止对阻塞概要信息的采样。

还有一个名叫`blockprofilerate`的包级私有变量，它是uint64类型的。这个变量的含义是，只要发现一个阻塞事事件的持续时间跨越了多少个 CPU 时钟周期，就可以对其进行采样。

另一方面，当我们需要获取阻塞概要信息的时候，需要先调用runtime/pprof包中的Lookup函数并传入参数值"block"，从而得到一个*runtime/pprof.Profile类型的值（以下简称Profile值）。在这之后，我们还需要调用这个Profile值的WriteTo方法，以驱使它把概要信息写进我们指定的写入器中。

**runtime/pprof.Lookup函数的调用方式是什么？**

`runtime/pprof.Lookup`函数（以下简称Lookup函数）的功能是，提供与给定的名称相对应的概要信息。这个概要信息会由一个Profile值代表。如果该函数返回了一个nil，那么就说明不存在与给定名称对应的概要信息。

`runtime/pprof`包已经为我们预先定义了 6 个概要名称。它们对应的概要信息收集方法和输出方法也都已经准备好了。我们直接拿来使用就可以了。

它们是：`goroutine、heap、allocs、threadcreate、block和mutex`。


**问题 4：如何为基于 HTTP 协议的网络服务添加性能分析接口？**

这个问题说起来还是很简单的。这是因为我们在一般情况下只要在程序中导入net/http/pprof代码包就可以了，就像这样：

```go
import _ "net/http/pprof"
```
然后，启动网络服务并开始监听，比如：
```go
log.Println(http.ListenAndServe("localhost:8082", nil))
```





