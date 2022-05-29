---
layout: post
title:  go的竞争检测
date: 2018-03-01 20:21:12
categories: 编程语言 go
tags:  编程语言 go
excerpt: go的竞争检测
---


`go run -race` 或者 `go build -race` 来进行竞争检测。

golang语言内部大概的实现就是同时开启多个goroutine执行同一个命令，并且纪录每个变量的状态。

```go 
package main

import(
    "time"
    "fmt"
)

func main() {
    a := 1
    go func(){
        a = 2
    }()
    a = 3
    fmt.Println("a is ", a)

    time.Sleep(2 * time.Second)
}

```

这个程序可以看出变量a出现了竞争。


在windows下执行
```sh
go run: -race and -msan are only supported on linux/amd64, freebsd/amd64, darwin
/amd64 and windows/amd64
```

在linux下执行
```go 
a is  3
==================
WARNING: DATA RACE
Write at 0x00c4200140a8 by goroutine 6:
  main.main.func1()
      /root/go/race.go:11 +0x3b

Previous write at 0x00c4200140a8 by main goroutine:
  main.main()
      /root/go/race.go:13 +0x8e

Goroutine 6 (running) created at:
  main.main()
      /root/go/race.go:10 +0x7d
==================
Found 1 data race(s)
exit status 66
```
**13行 出现变量竞争。**