---
layout: post
title: go 笔记 并发编程
date: 2018-03-03 07:21:12
categories:  go 
tags: go
excerpt: go 并发编程,最主要就是 select。
---


在 Go 语言中最重要的一个特性，那就是 go 关键字。

# 一、并发基础

并发的实现有哪些：

- **多进程**。多进程是在操作系统层面进行并发的基本模式。
- **多线程**。多线程在大部分操作系统上都属于系统层面的并发模式。
- **基于回调的非阻塞/异步IO**。在很多 高并发服务器开发实践中，使用多线程模式会很快耗尽服务器的内存和CPU资源。
- **协程**。本质上是一种用户态线程，不需要操作系统来进行抢占式调度， 且在真正的实现中寄存于线程中，因此，系统开销极小，可以有效提高线程的任务并发性，而避免多线程的缺点。

我们习惯于串行模式，串行方式有确定性，但是，为了提高处理效率，我们引入了并发模式。 并发模式会存在不确定性，导致处理行为的意外发生，所以我们采用到共享内存的方式。为了保证共享内存的有效性，我们采取了很多措施，比如加锁等，来避免死锁或资源竞争。实践证明，我们很难面面俱到，往往会在工程中遇到各种奇怪的故障和问题。

所以，又研究另一种新的系统模型，称之为“**消息传递系统**”。

# 二、协程

协程，是一种轻量级线程。之后叫“轻量级”是因为它可以轻松创建上百万个而不会导致系统资源衰竭。而线程和进程通常最多也不能超过1万个。

Go 语言在语言级别支持轻量级线程，叫 goroutine。

Go 语言标准库提供的所有系统调用操作 (当然也包括所有同步 IO 操作)，都会出让 CPU 给其他goroutine。这让事情变得非常简单，让轻
量级线程的切换管理不依赖于系统的线程和进程，也不依赖于CPU的核心数量。

# 三、goroutine

goroutine 是 Go 语言中的轻量级线程实现，由Go运行时(runtime)管理。

在一个函数调用前加上go关键字，这次调用就会在一个新的goroutine中并发执行。

如：

```go 
package main

import "fmt"

func Add(x, y int) { z := x + y
        fmt.Println(z)
}

func main() {
    for i := 0; i < 10; i++ {
        go Add(i, i) 
    }
}
```

Go 程序从初始化 main package 并执行 main() 函数开始，当 main() 函数返回时，程序退出， 且程序并不等待其他goroutine(非主goroutine)结束。把这个例子什么也不会输出。

# 四、并发通信

从上面的例子，可以看到我们需要处理的是并发之间的通信。 

有两种最常见的并发通信模型:共享数据和消息。

共享数据是指多个并发单元分别保持对同一个数据的引用，实现对该数据的共享。被共享的数据可能有多种形式，比如内存数据块、磁盘文件、网络数据等。

```go
package main
import "fmt"
import "sync"
import "runtime"

var counter int = 0

func Count(lock *sync.Mutex) { lock.Lock()
    counter++
    fmt.Println(z)
    lock.Unlock()
}

func main() {
    lock := &sync.Mutex{}

    for i := 0; i < 10; i++ { 
        go Count(lock)
    }

    for { 
        lock.Lock()
        c := counter
        lock.Unlock()
        runtime.Gosched() 
        if c >= 10 {
            break
        } 
    }
}
```

这种方式是引进了锁的方式来完成通信，这个试比较晦涩，如C++ 相同的方式， 并没有什么太大的优势。 

Go 语言提供的是另一种以消息机制的通信模型，被称为 channel。

# 五、channel

channel 是 Go 语言在语言级别提供的 goroutine 间的通信方式。 我们可以使用channel在两个或 多个goroutine之间传递消息。

channel 是**进程内**的通信方式，因此通过 channel 传递对象的过程和调用函数时的参数传递行为比较一致，比如也可以传递指针等。

如果需要跨进程通信，我们建议用分布式系统的方法来解决，比如使用 Socket 或者 HTTP 等通信协议。Go 语言对于网络方面也有非常完善的支持。

我们可以用 channel 重写上例子。

```go 
package main
import "fmt"

func Count(ch chan int) { 
    ch <- 1
    fmt.Println("Counting")
}

func main() {

    chs := make([]chan int， 10) 
    for i := 0; i < 10; i++ {
        chs[i] = make(chan int)
        go Count(chs[i]) 
    }

    for _, ch := range(chs) { 
        <-ch
    }
}
```
义了一个包含10个 channel 的数组(名为chs)，并把数组中的每个 channel 分配给10个不同的 goroutine。

在每个 goroutine 的Add()函数完成后，我们通过`ch <- 1`语句向对应的 channel 中写入一个数据。在这个 channel 被读取前，这个操作是阻塞的。在所有的 goroutine 启动完成后，我们通过 `<-ch` 语句从10个 channel 中依次读取数据。

## 5.1 基本语法

channel 的声明形式为:

```go
var chanName chan ElementType
```

lementType指定这个 channel所能传递的元素类型。如：

```go 
var ch chan int
var m map[string] chan bool
```

定义一个channel也很简单，直接使用内置的函数 make() 即可

```go
ch := make(chan int)
```

**在 channel 的用法中，最常见的包括写入和读出。**

写入，将一个数据写入(发送)至channel：
```go
ch <- value
```
向channel写入数据通常会导致程序阻塞。直到有其他goroutine从这个channel中读取数据。

```go
value := <-ch
```
如果channel之前没有写入数据，那么从channel中读取数据也会导致程序阻塞，直到channel 中被写入数据为止。

## 5.2 select

Go 语言直接在语言级别支持 select 关键字，用于处理异步 IO 问题。

select 的用法与 switch 语言非常类似，由 select 开始一个新的选择块，每个选择条件由 case 语句来描述。

最大的一条限制就是每个 case 语句里必须是一个IO操作。

```go
select {
    case <-chan1:
    // 如果chan1成功读到数据，则进行该case处理语句 
    case chan2 <- 1:
    // 如果成功向chan2写入数据，则进行该case处理语句 
    default:
    // 如果上面都没有成功，则进入default处理流程 }
```

## 5.3 缓冲机制

当我们需要持续传输大量数据时，channel 就要喧上缓存。

创建一个带缓冲的channel:

```go
 c := make(chan int, 1024)
```
在调用 make() 时将缓冲区大小作为第二个参数传入即可，比如上面这个例子就创建了一个大小 为 1024 的 int 类型 channel，即使没有读取方，写入方也可以一直往channel里写入，在缓冲区被 填完之前都不会阻塞。

## 5.4 超时机制

在并发编程的通信过程中，最需要处理的就是超时问题。即向 channel 写数据时发现 channel已满，或者从 channel 试图读取数据时发现 channel 为空。如果不正确处理这些情况，很可能会导致整个goroutine锁死。

Go 语言没有提供直接的超时处理机制，但我们可以利用 select 机制。select 的特点是只要其中一个 cas e已经完成，程序就会继续往下执行，而不会考虑其他 case 的情况。    

```go
// 首先，我们实现并执行一个匿名的超时等待函数 
timeout := make(chan bool, 1)
go func() {
    time.Sleep(1e9) // 等待1秒钟
    timeout <- true
}()

// 然后我们把timeout这个channel利用起来
select {
    case <-ch:
    // 从ch中读取到数据
    case <-timeout:
    // 一直没有从ch中读取到数据，但从timeout中读取到了数据
}
```

这样使用 select 机制可以避免永久等待的问题，因为程序会在 timeout 中获取到一个数据 后继续执行，无论对 ch 的读取是否还处于等待状态，从而达成1秒超时的效果。

## 5.5 channel 的传递

在 Go 语言中 channel 本身也是一个原生类型，与 map类型地位一样，因此 channel 本身在定义后也可以通过 channel 来传递。

## 5.6 单向 channel

顾名思义，单向channel只能用于发送或者接收数据。

单向channel变量的声明非常简单，如下:

```go
var ch1 chan int        // ch1是一个正常的channel，不是单向的 
var ch2 chan<- float64  // ch2是单向channel，只用于写float64数据
var ch3 <-chan int      // ch3是单向channel，只用于读取int数据
```
基于 ch4，我们通过类型转换初始化了两个单向 channel:单向读的 ch5 和单向写的 ch6。

单向 channel 也是起到这样的一种契约作用。

## 5.7 关闭 channel

关闭channel非常简单，直接使用Go语言内置的close()函数即可。

```go
close(ch)
```

如何判断一个channel是否已经被关 闭?我们可以在读取的时候使用多重返回值的方式:

```go 
 x, ok := <-ch
```
这个用法与 map 中的按键获取 value 的过程比较类似，只需要看第二个 bool 返回值即可，如果返回值是false则表示ch已经被关闭。

# 六、多核并行化

当前版本的 Go 编译器还不能很智能地去发现和利用多核的优势。虽然 我们确实创建了多个 goroutine。

实际上所有 goroutine 都可能运行在同一个 CPU 核心上，在一个 goroutine 得到时间片执行的时候，其他 goroutine 都会处于等待状态。

在Go语言升级到默认支持多CPU的某个版本之前，我们可以先通过设置环境变量 GOMAXPROCS 的值来控制使用多少个 CPU 核心。具体操作方法是通过直接设置环境变量 GOMAXPROCS 的值，或者在代码中启动 goroutine 之前先调用以下这个语句以设置使用16个 CPU 核心:

```go
runtime.GOMAXPROCS(16)
```

# 七、出让时间片

我们可以在每个 goroutine 中控制何时主动出让时间片给其他 goroutine，这可以使用 runtime 包中的 Gosched() 函数实现。

# 八、同步

Go 语言的设计者虽然对channel有极高的期望，但也提供了妥善的资源锁方案。

## 8.1  同步锁

Go语言包中的 sync 包提供了两种锁类型: sync.Mutex 和 sync.RWMutex。Mutex 是最简单 的一种锁类型，

## 8.2 全局唯一性操作

于从全局的角度只需要运行一次的代码，比如全局初始化操作，Go语言提供了一个 Once 类型来保证全局的唯一性操作。
