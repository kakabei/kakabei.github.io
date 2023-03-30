---
layout: post
title: golang init 函数
date: 2019-03-30 09:21:12
categories:  golang 
tags:   golang
excerpt: golang 有一个特殊的函数 `init `函数，先于`main`函数执行，作用实现包的一些初始化操作
---

golang 有一个特殊的函数 `init `函数，先于`main`函数执行，作用实现包的一些初始化操作。 

# init 函数的特点： 

- init 函数先于 main 函数自动执行， 不能被其他函数调用。
- init 函数没有输入参数和返回值。
- 每个包可以有多个init 函数
- 包的每个源文件也可以有多个init 函数。
- 同一个包的init 执行顺序，golang 没有明确的定义，所以要逻辑注意不要依赖这个执行顺序。
- 不同包的init 函数按照包饿的依赖关系决定挂靠顺顺序。

# golang 初始化顺序

初始化顺序： 变量初始化->` init()`-> `main()`

如：

```go
package main

import "fmt"

var i int = initVer()

func init() {
	fmt.Println("0 : init......")
}

func initVer() int {
	fmt.Println("0 : initVer.....")
	return 100
}

func main() {
	fmt.Println("1 : main......")
}

```



