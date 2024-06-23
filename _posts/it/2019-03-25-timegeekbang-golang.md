---
layout: post
title: 《极客时间》go语言核心36讲	
date: 2019-03-25 09:21:12
categories: 技术学习笔记
tags:  golang 极客时间 
excerpt: sync.WaitGroup类型（以下简称WaitGroup类型）是开箱即用的，也是并发安全的
---

# go语言核心36讲 sync.WaitGroup 和 sync.Once

`sync.WaitGroup` 类型是开箱即用的，也是并发安全的。同时，它一旦被真正使用就不能被复制了。

WaitGroup 类型拥有三个指针方法：Add、Done 和 Wait。你可以想象该类型中有一个计数器，它的默认值是0。我们可以通过调用该类型值的Add方法来增加，或者减少这个计数器的值。

一般情况下，会用这个方法来记录需要等待的 goroutine 的数量。相对应的，这个类型的 Done 方法，用于对其所属值中计数器的值进行减一操作。我们可以在需要等待的 goroutine 中，通过 defer 语句调用它。


而此类型的 Wait 方法的功能是，阻塞当前的 goroutine，直到其所属值中的计数器归零。如果在该方法被调用的时候，那个计数器的值就是 0，那么它将不会做任何事情。


`sync.WaitGroup` 类型值中计数器的值可以小于0吗？

这里的典型回答是：不可以。


另外，你可能已经知道，WaitGroup 值是可以被复用的，但需要保证其计数周期的完整性。这里的计数周期指的是这样一个过程：该值中的计数器值由 0 变为了某个正整数，而后又经过一系列的变化，最终由某个正整数又变回了0。

也就是说，只要计数器的值始于0又归为0，就可以被视为一个计数周期。在一个此类值的生命周期中，它可以经历任意多个计数周期。但是，只有在它走完当前的计数周期之后，才能够开始下一个计数周期。

![](/assets/timegeekbang/go-sync.png) 

不要把增加其计数器值的操作和调用其Wait方法的代码放在相同的 goroutine 中执行。换句话说，要杜绝对同一个 WaitGroup 值的两种操作的并发执行。


问题：sync.Once 类型值的 Do 方法是怎么保证只执行参数函数一次的？

与 `sync.WaitGroup` 类型一样，sync.Once 类型也属于结构体类型，同样也是开箱即用和并发安全的。由于这个类型中包含了一个 sync.Mutex 类型的字段，所以，复制该类型的值也会导致功能的失效。


扩展：https://www.cnblogs.com/linyihai/p/10285437.html

# go语言核心36讲 原子操作


在众多的同步工具中，真正能够保证原子性执行的只有原子操作（atomic operation）。原子操作在进行的过程中是不允许中断的。在底层，这会由 CPU 提供芯片级别的支持。

**sync/atomic包中提供了几种原子操作？可操作的数据类型又有哪些？**

sync/atomic包中的函数可以做的原子操作有：加法（add）、比较并交换（compare and swap，简称 CAS）、加载（load）、存储（store）和交换（swap）。

**第一个衍生问题 ：**我们都知道，传入这些原子操作函数的第一个参数值对应的都应该是那个被操作的值。比如，atomic.AddInt32函数的第一个参数，对应的一定是那个要被增大的整数。可是，这个参数的类型为什么不是int32而是*int32呢？

回答是：因为原子操作函数需要的是被操作值的指针，而不是这个值本身；被传入函数的参数值都会被复制，像这种基本类型的值一旦被.传入函数，就已经与函数外的那个值毫无关系了。

所以，传入值本身没有任何意义。unsafe.Pointer类型虽然是指针类型，但是那些原子操作函数要操作的是这个指针值，而不是它指向的那个值，所以需要的仍然是指向这个指针值的指针。

**第二个衍生问题：** 用于原子加法操作的函数可以做原子减法吗？比如，atomic.AddInt32函数可以用于减小那个被操作的整数值吗？

回答是：当然是可以的。atomic.AddInt32函数的第二个参数代表差量，它的类型是int32，是有符号的。如果我们想做原子减法，那么把这个差量设置为负整数就可以了。

对于atomic.AddUint64函数做原子减法，就不能这么直接了，因为它们的第二个参数的类型分别是uint32和uint64，都是无符号的。所以要做转换 uint32(int32(-3))。如果不这么做的话，可报错。还有一种更加考虑直接的方式可以传入^uint32(-N-1)

```go
atomic.AddUint32(*addr,uint32(int32(n)))
//或
atomic.AddUint32(&addr,^uint32(-n-1))
```

**第三个衍生问题：** 比较并交换操作与交换操作相比有什么不同？优势在哪里？

回答是：比较并交换操作即 CAS 操作，是有条件的交换操作，只有在条件满足的情况下才会进行值的交换。
所谓的交换指的是，把新值赋给变量，并返回变量的旧值。


**第四个衍生问题：** 假设我已经保证了对一个变量的写操作都是原子操作，比如：加或减、存储、交换等等，，还有必要使用原子操作吗？

回答是：很有必要。如果写操作还没有进行完，读操作就来读了，那么就只能读到仅修改了一部分的值。这显然破坏了值的完整性，读出来的值也是完全错误的。

** 问题：怎样用好sync/atomic.Value?**

atomic.Value类型是开箱即用的，我们声明一个该类型的变量（以下简称原子变量）之后就可以直接使用了。这个类型使用起来很简单，它只有两个指针方法：Store和Load。不过，虽然简单，但还是有一些值得注意的地方的。

1 一旦被真正的使用，就不应该被复制，atomic.Value类型属于结构体类型， 而结构体类型属于值类型。复制时会产生个一完全分离的新值。

2 不能用原子偷走一存储nil

3 向原子值存储的第一个值，决定了它今后能且只能在座哪一个类型的值。即使使用接口类型，然后再存储这个接口的某个实现类型的值，还是不可以的。

**使用建议：**

1 不要把内部使用的原子值暴露给外界

2 如果不和地不让包外，或模块外的代码使用你的原子值，那么可以声明一个包级私有的原子变量，然后再通过一个或多个公开的函数让外界间接地信息使用它。注意，这种情况下不要把原子值传递外界，不论是传递原子值本身还是它的指针值。

3 如果通过某个函数可以向内部民的原子值 存储的话，那么就应该在这个函数中先判断被存储值类型的合法性。

4 最好把原子值封装到一个数据类型中，比如结构体类型。

尽量不要向原子值中存储引用类型的值，容易造成安全漏洞。

扩展阅读：https://www.jianshu.com/p/228c119a7d0e

# go语言核心36讲 互斥锁 sync.Mutex 与 sync.RWMutex


一个互斥锁可以被 用来保护一个临界区或一组相关临界区。保证在同一时刻只有一个Goroutine处于该临界区之内

为了兑现这保证上，每当goroutine 想进入临界区时，都要先对它进行锁定 ，离开时临界区时都要及时地对它进行解锁。

```go
mu.Lock()
_, err := writer.Write([]byte(data))
if err != nil {
 log.Printf("error: %s [%d]", err, id)
}
mu.Unlock()
```

**使用互斥锁时有哪 些注意事项：**

1 不要重复锁定互斥锁。

2 不要忘记解锁互斥锁，必要时用 defer 语句 

3 不要对沿未锁定或者已经解锁的互斥锁解锁

4 不要在多个函数之前直接传递互斥锁

死锁时抛出的panic是属于致使错误，都是无法被恢复的，调用recover函数对它们起不了任何作用。

互斥锁是开箱即用的。sysnc.Mutex类型 是一个结构体类型，属于值类型中的一种，把它传给一个函数、将它从函数中返回，把它赋给其他变量。让它进入 某个通道都会导致它的副本的产生。
它们是独立的，都是不同的互斥锁。


**读写锁与互斥锁有哪 异同？**

sync.RWMute类型的值代表。 都是开箱即用。 它是把对共享资源的“读操作”和“写操作”区别对待。

比互斥锁有列加细腻的访问控制。

一个读写锁中实际上包含了两个锁，即：读锁和写锁。sync.RWMutex类型中的Lock方法和Unlock方法分别用于对写锁进行锁定和解锁，而它的RLock方法和RUnlock方法则分别用于对读锁进行锁定和解锁。


**另外，对于同一个读写锁来说有如下规则：**

1 在写锁已被锁定的情况下再试图锁定写锁，会阻塞当前的 goroutine。

2 在写锁已被锁定的情况下试图锁定读锁，也会阻塞当前的 goroutine

3 在读锁已被锁定的情况下试图锁定写锁，同样会阻塞当前的goroutine。

4 在读锁已被锁定的情况下再试图锁定读锁，并不会阻塞当前的goroutine。

换一个角度来说，对于某个受到读写锁保护的共享资源，多个写操作不能同时进行，写操作和读操作也不能同时进行，但多个读操作却可以同时进行。


go 语言代码  实例：
```go
package main

import (
    "bytes"
    "errors"
    "fmt"
    "io"
    "log"
    "sync"
    "time"
)

// singleHandler 代表单次处理函数的类型。
type singleHandler func() (data string, n int, err error)

// handlerConfig 代表处理流程配置的类型。
type handlerConfig struct {
    handler   singleHandler // 单次处理函数。
    goNum     int           // 需要启用的goroutine的数量。
    number    int           // 单个goroutine中的处理次数。
    interval  time.Duration // 单个goroutine中的处理间隔时间。
    counter   int           // 数据量计数器，以字节为单位。
    counterMu sync.Mutex    // 数据量计数器专用的互斥锁。

}

// count 会增加计数器的值，并会返回增加后的计数。
func (hc *handlerConfig) count(increment int) int {
    hc.counterMu.Lock()
    defer hc.counterMu.Unlock()
    hc.counter += increment
    return hc.counter
}

func main() {
    // mu 代表以下流程要使用的互斥锁。
    // 在下面的函数中直接使用即可，不要传递。
    var mu sync.Mutex

    // genWriter 代表的是用于生成写入函数的函数。
    genWriter := func(writer io.Writer) singleHandler {
        return func() (data string, n int, err error) {
            // 准备数据。
            data = fmt.Sprintf("%s\t",
                time.Now().Format(time.StampNano))
            // 写入数据。
            mu.Lock()
            defer mu.Unlock()
            n, err = writer.Write([]byte(data))
            return
        }
    }

    // genReader 代表的是用于生成读取函数的函数。
    genReader := func(reader io.Reader) singleHandler {
        return func() (data string, n int, err error) {
            buffer, ok := reader.(*bytes.Buffer)
            if !ok {
                err = errors.New("unsupported reader")
                return
            }
            // 读取数据。
            mu.Lock()
            defer mu.Unlock()
            data, err = buffer.ReadString('\t')
            n = len(data)
            return
        }
    }

    // buffer 代表缓冲区。
    var buffer bytes.Buffer

    // 数据写入配置。
    writingConfig := handlerConfig{
        handler:  genWriter(&buffer),
        goNum:    5,
        number:   4,
        interval: time.Millisecond * 100,
    }
    // 数据读取配置。
    readingConfig := handlerConfig{
        handler:  genReader(&buffer),
        goNum:    10,
        number:   2,
        interval: time.Millisecond * 100,
    }

    // sign 代表信号的通道。
    sign := make(chan struct{}, writingConfig.goNum+readingConfig.goNum)

    // 启用多个goroutine对缓冲区进行多次数据写入。
    for i := 1; i <= writingConfig.goNum; i++ {
        go func(i int) {
            defer func() {
                sign <- struct{}{}
            }()
            for j := 1; j <= writingConfig.number; j++ {
                time.Sleep(writingConfig.interval)
                data, n, err := writingConfig.handler()
                if err != nil {
                    log.Printf("writer [%d-%d]: error: %s",
                        i, j, err)
                    continue
                }
                total := writingConfig.count(n)
                log.Printf("writer [%d-%d]: %s (total: %d)",
                    i, j, data, total)
            }
        }(i)
    }

    // 启用多个goroutine对缓冲区进行多次数据读取。
    for i := 1; i <= readingConfig.goNum; i++ {
        go func(i int) {
            defer func() {
                sign <- struct{}{}
            }()
            for j := 1; j <= readingConfig.number; j++ {
                time.Sleep(readingConfig.interval)
                var data string
                var n int
                var err error
                for {
                    data, n, err = readingConfig.handler()
                    if err == nil || err != io.EOF {
                        break
                    }
                    // 如果读比写快（读时会发生EOF错误），那就等一会儿再读。
                    time.Sleep(readingConfig.interval)
                }
                if err != nil {
                    log.Printf("reader [%d-%d]: error: %s",
                        i, j, err)
                    continue
                }
                total := readingConfig.count(n)
                log.Printf("reader [%d-%d]: %s (total: %d)",
                    i, j, data, total)
            }
        }(i)
    }

    // signNumber 代表需要接收的信号的数量。
    signNumber := writingConfig.goNum + readingConfig.goNum
    // 等待上面启用的所有goroutine的运行全部结束。
    for j := 0; j < signNumber; j++ {
        <-sign
    }
}
```
# go语言核心36讲 条件变量 sync.Cond


条件变量是和互斥锁一起使用的。
条件变量是和互斥锁一起使用的。

条件变量并不是被用来保护临界区和共享资源的，它是用于协调想要访问共享资源的那些线程的。当共享资源的状态发生变化时，它可以被用来通知被互斥锁阻塞的线程。

条件变量提供的方法有三个：等待通知（wait）、单发通知（signal）和广播通知（broadcast）。

这里有一个疑问：
```go
var mailbox uint8
var lock sync.RWMutex
sendCond := sync.NewCond(&lock)
recvCond := sync.NewCond(lock.RLocker())
```

endCond := sync.NewCond(&lock)和recvCond := sync.NewCond(lock.RLocker()) 

传入的一个是 &lock 另一个是 lock.RLocker() 为什么？


**条件变量的Wait方法做了什么？**

条件变量的Wait方法主要做了四件事。

1　把调用它的 goroutine（也就是当前的 goroutine）加入到当前条件变量的通知队列中。

2　解锁当前的条件变量基于的那个互斥锁。

3　让当前的 goroutine 处于等待状态，等到通知到来时再决定是否唤醒它。此时，这个 goroutine 就会阻塞在调用这个Wait方法的那行代码上。

4　如果通知到来并且决定唤醒这个 goroutine，那么就在唤醒它之后重新锁定当前条件变量基于的互斥锁。自此之后，当前的goroutine 就会继续执行后面的代码了。

**条件变量的Signal方法和Broadcast方法有哪些异同？**

条件变量的Signal方法和Broadcast方法都是被用来发送通知的，不同的是，前者的通知只会唤醒一个因此而等待的 goroutine，而后者的通知却会唤醒所有为此等待的 goroutine。

扩展 ：https://blog.csdn.net/wentyoon/article/details/81174288

# 《极客时间》go语言核心36讲 程序性能分析基本



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










