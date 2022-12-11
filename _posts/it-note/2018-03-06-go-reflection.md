---
layout: post
title: go 笔记 反射
date: 2018-03-06 07:21:12
categories:  go 
tags: go
excerpt: go 通过反射，你可以获取丰富的类型信息，并可以利用这些类型信息做非常灵活的工作。
---

反射( reflection )是在 Java 出现后迅速流行起来的一种概念。通过反射，你可以获取丰富的类型信息，并可以利用这些类型信息做非常灵活的工作。

# 一、基本概念

Type 和 Value，它们也是 Go 语言包中 reflect 空间里最重要的两个类型。

对所有接口进行反射，都可以得到一个包含 Type 和 Value 的信息结构。

如：

```go 
var reader io.Reader
reader = &MyReader{"a.txt"}
```
对上例的 reader 进行反射，也将得到一个 Type 和 Value。 

Type 为 `io.Reader`，Value 为 `MyReader{"a.txt"}`。

顾名思义，Type主要表达的是被反射的这个变量本身的类型信息，而Value则为该变量实例本身的信息。

# 二、基本用法

## 获取类型信息

```go
package main
import ( 
    "fmt"
    "reflect" )

func main() {
    var x float64 = 3.4 
    fmt.Println("type:", reflect.TypeOf(x))
}
```

以上代码将输出如下的结果: `type: float64`

## 获取值类型

类型 Type 中有一个成员函数 `CanSet()` 。 Go语言中所有的类型都是值类型。在 调用 `ValueOf()` 的地方，需要注意到 x 将会产生一个副本，因此 `ValueOf()`内部对 x 的操作其实 都是对着 x 的一个副本。

```go 
package main

import (
	"fmt"
	"reflect"
)

func main() {

	var x float64 = 3.4
	p := reflect.ValueOf(&x) // 注意:得到X的地址
	fmt.Println("type of p:", p.Type())
	fmt.Println("settability of p:", p.CanSet())
	v := p.Elem()
	fmt.Println("settability of v:", v.CanSet())
	v.SetFloat(7.1)
	fmt.Println(v.Interface())
	fmt.Println(x)
}
```

可以通过 `Elem()` 让值成可设属性。

# 三、对结构的反射操作

对于结构的反射操作并没有根本上的不同，只是用了 `Field()` 方法来按索引获取 对应的成员。
在试图修改成员的值时，也需要注意可赋值属性。
```go 
type T struct { 
    A int
    B string 
}

t := T{203, "mh203"}
s := reflect.ValueOf(&t).Elem() 
typeOfT := s.Type()

for i := 0; i < s.NumField(); i++ {
        f := s.Field(i)
        fmt.Printf("%d: %s %s = %v\n", i,typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```