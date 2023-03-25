---
layout: post
title: golang 编译时传递参数
date: 2018-04-10 09:21:12
categories: go 
tags:  go
excerpt: go 编译时传递参数
---



很有意思，可能运用的场景不多。如果想调试某个模块时，可以在编译直接给他赋值。

```go
package main

import "fmt"

var BuildDate = "no build date"

func main() {
    fmt.Printf("build date: %s\n", BuildDate)
}
```

传递参数步骤：

```sh 
$ go build demo.go
$ ./demo
build date: no build date
$ DATE=`date '+%Y-%m-%d-%I:%M:%S'`
$ go build -ldflags "-X main.BuildDate=$DATE" demo.go
$ ./demo
build date: 2018-05-03-03:15:35
```
通过-X选项，可以给go程序传递相关参数。例如示例中，借助-X选项，将编译日期BuildDate实时传递到程序中。另外，如版本信息之类也可以通过该方式实现。
