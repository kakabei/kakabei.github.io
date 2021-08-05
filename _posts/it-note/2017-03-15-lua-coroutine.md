---
layout: post
title: lua 协程
date: 2017-03-15 21:12:15
categories: 编程语言 lua
tags: lua   
excerpt: lua coroutine 一种多任务的方式。
---

>协同程序（coroutine）与多线程情况下的线程比较类似：有自己的堆栈，自己的局
部变量，有自己的指令指针，但是和其他协同程序共享全局变量等很多信息。线程和协
同程序的主要不同在于：在多处理器情况下，从概念上来讲多线程程序同时运行多个线
程；而协同程序是通过协作来完成，在任一指定时刻只有一个协同程序在运行，并且这
个正在运行的协同程序只有在明确的被要求挂起的时候才会被挂起。

宏观上可能看成是比线程更小的执行单位。但本质上，它是线程管理下的单位。这就是上面所说的在任一指定时刻只有一个协同程序在运行。是一种多任务方式。

### 协同的基础

lua 提供对应一些协程的函数。如 create() status() resume() 等等。

创建一个协程create()

```lua
co = coroutine.create(function ()
print("hello world!")
end)

print(co) 
```

查看状态，可用status

```lua
print(coroutine.status(co)) --> suspended
```

协同有三个状态：挂起态、运行态、停止态。

让一个协程从挂起状变为运行态

```lua
coroutine.resume(co)
```

让一个协程挂起，可用yield()

```lua
co = coroutine.create(function ()
for i=1,10 do
	print("co", i)
	coroutine.yield()
	end
end)
```
在程序使用resume后，协程被激活。

看 resume-yield 可以相互交换数据。

1 非对称的情况：

resume 把参数传入协程里。

```lua
co = coroutine.create(function (a,b,c)
print("co", a,b,c)
end)
coroutine.resume(co, 1, 2, 3) --> co 1 2 3
```

2 对称的情况：

yield 会把值返还给resume。

```lua
co = coroutine.create(function (a,b)
coroutine.yield(a + b, a - b)
end)
print(coroutine.resume(co, 20, 10)) --> true 30 10
```