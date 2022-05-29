---
layout: post
title: 《极客时间》go语言核心36讲 互斥锁 sync.Mutex与sync.RWMutex
date: 2019-03-27 09:21:12
categories:  go
tags:  极客时间笔记
excerpt: 一个互斥锁可以被,用来保护一个临界区或一组相关临界区
---


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


