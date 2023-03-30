---
layout: post
title: 《极客时间》go语言核心36讲 条件变量 sync.Cond
date: 2019-03-28 09:21:12
categories:  golang
tags:  golang 极客时间 技术学习笔记
excerpt: 条件变量是和互斥锁一起使用的。
---


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