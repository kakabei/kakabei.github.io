---
layout: post
title: golang 笔记 面向对象编程
date: 2018-03-02 07:21:12
categories:  golang 
tags: golang
excerpt: go 在面各对象编程中最重要的就是接口这个知识点了。
---

# 一、类型系统

类型系统是指一个语言的类型体系结构。通常包含如下基本 内容:

- 基础类型，如： byte、int、bool、float等。
- 复合类型，如： 数组、结构体、指针等。
- 可以指向任意对象的类型(Any类型)。
- 值语义和引用语义。
- 面向对象，即所有具备面向对象特征(比如成员方法)的类型。
- 接口。

## 为类型添加方法

在 golang 语言中，除了指针类型，可以给任意类型添加相应的方法.

```go
type Integer int
func (a Integer) Less(b Integer) bool { 
    return a < b
}
```

定义了一个新类型 Integer，它和 int 没有本质不同，只是它为内置的 int 类型增加了个新方法 Less()。

golang 语言中的面向对象最为直观，也无需支付额外的成本。如果要求对象必须以指针传递， 这有时会是个额外成本，因为对象有时很小(比如4字节)，用指针传递并不划算。只有在你需要修改对象的时候，才必须用指针。

如：

```go
func (a *Integer) Add(b Integer) { 
    *a += b
 }
```

会改变 a 的值。 

```go
func (a Integer) Add(b Integer) { 
    a += b
}
```

不会改变 a 的值。 

因为 golang 语言和 C 语言一样，类型都是基于**值传递**的。要想修改变量的值，只能传递指针。

## 值语义和引用语义

值语义和引用语义的差别在于赋值，比如下面的例子:

```go 
b = a b.Modify()
```

如果 b 的修改不会影响 a 的值，那么此类型属于值类型。如果会影响 a 的值，那么此类型是引用类型。

golang 语言中的大多数类型都基于值语义，包括:

- 基本类型，如： byte、int、bool、float32、float64和string等。
- 复合类型，如： 数组(array)、结构体(struct)和指针(pointer)等。

Go语言中的数组和基本类型没有区别，是很纯粹的值类型。要想表达引用，需要用指针。

Go语言中有 4 个类型是引用类型

- 数组切片: 指向数组(array)的一个区间。
- map: 极其常见的数据结构，提供键值查询能力。
- channel: 执行体(goroutine)间的通信设施。
- 接口(interface):对一组满足某个契约的类型的抽象。

## 结构体

golang 语言的结构体(struct)和其他语言的类(class)有同等的地位，但 golang 语言放弃了包括继承在内的大量面向对象特性，只保留了**组合**(composition)这个最基础的特性。

所有的 golang 语言类型(指针类型除外)都可以有自己的方法。结构体只是很普通的复合类型，平淡无奇。

例如，我们要定义一个矩形类型:
```go
type Rect struct { 
    x, y float64
    width, height float64
}
```
然后我们定义成员方法 `Area()` 来计算矩形的面积:

```go 
func (r *Rect) Area() float64 { 
    return r.width * r.height
}
```
可以看出， golang 语言中结构体的使用方式与 C 语言并没有明显不同。

# 二、初始化

在 golang 语言中，未进行显式初始化的变量都会被初始化为该类型的零值，例如 bool 类型的零 值为 false，int 类型的零值为 0，string 类型的零值为空字符串。

在 golang 语言中没有构造函数的概念。对象的创建通常交由一个全局的创建函数来完成，以 `NewXXX` 来命名，表示“构造函数”。

```go 
func NewRect(x, y, width, height float64) *Rect { 
    return &Rect{x, y, width, height}
}
```

# 三、 匿名组合

golang 语言也提供了继承，但是采用了组合的方法，所以我们将其称为匿名组合。

```go
type Base struct { 
    Name string
}

func (base *Base) Foo() { ... }
func (base *Base) Bar() { ... }

type Foo struct { 
    Base
    // ... 
}

func (foo *Foo) Bar() { 
    foo.Base.Bar()
    // ... 
}
```

定义了一个 Base 类, 实现了 Foo() 和 Bar () 两个成员方法，然后定义了一个 Foo 类，该类从 Base 类“继承”并改写了 Bar() 方法, 该方法实现时先调用了基类的Bar() 方法。

```go
type Foo struct { 
    *Base
    // ... 
}
```
代码仍然有“派生”的效果，只是 Foo 创建实例的时候，需要外部提供一个 Base 类 实例的指针。

组合的类型有重名时，外部的名称会隐藏掉内部的名称。

```go 
type X struct { 
    Name string
}

type Y struct { 
    X
    Name string 
}
```

X 中的 Name 会被隐藏。

# 四、可见性

要使某个符号对其他包( package )可以访问，需要将该符号定义为以大写字母开头。

```go

type Rect struct { 
    X, Y float64
    Width, Height float64 
}
```

Rect 类型的成员变量就全部被导出了，可以被所有其他引用了 Rect 所在包的代码访问到。

# 五、接口

## 定义接口

关键字 interface 用来定义接口：

```go
type interface_name interface {
   method_name1([args ...arg_type]) [return_type]
   method_name2([args ...arg_type]) [return_type]
   method_name3([args ...arg_type]) [return_type]
   //...
   method_namen([args ...arg_type]) [return_type]
}
```

## 实现接口

一个结构体实现了某个接口的所有方法，则此结构体就实现了该接口。

```go

// 接口
type IFile interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
    Seek(off int64, whence int) (pos int64, err error) Close() error
}

// File 实现接口
type File struct { 
    // ...
}
func (f *File) Read(buf []byte) (n int, err error)
func (f *File) Write(buf []byte) (n int, err error)
func (f *File) Seek(off int64, whence int) (pos int64, err error)
func (f *File) Close() error
```

File 实现了 IFile 的所有方法，即表式 结构体 File 实现了 IFile 的所有接口。

## Any 类型 

任何对象实例都满足空接口 interface{}，所以 interface{} 看起来像是可以指向任何对象的Any类型.

空接口 ：`interface{}`

不包含任何的方法，正因为如此，所有的类型都实现了空接口，因此空接口可以存储任意类型的数值。

fmt 包下的 Print 系列函数，其参数大多是空接口类型，也可以说支持任意类型：

```go 
func Print(a ...interface{}) (n int, err error)
func Println(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```
```go 
// 定义一个空接口
type Empyt_interface interface {
}

// 定义一个入参为任意类型的函数
func getInfo(arg Empyt_interface) {
    fmt.Println("getInfo 函数.....", arg)
}
// 也可以写成如下形式，更推荐
func getInfo2(arg interface{}) {
    fmt.Println("getInfo2 函数.....", arg)
}
```

## 接口嵌套

接口嵌套就是一个接口中包含了其他接口，如果要实现外部接口，那么就要把内部嵌套的接口对应的所有方法全实现了。

如：

```go 
// 定义3个接口
type A interface {
    test1()
}

type B interface {
    test2()
}

// 定义嵌套接口
type C interface {
    A
    B
    test3()
}

type Person struct {
    // 如果想实现接口C，那不止要实现接口C的方法，还要实现接口A，B中的方法
}

func (p Person) test1() {
    fmt.Println("test1 方法................")
}

func (p Person) test2() {
    fmt.Println("test2 方法................")
}

func (p Person) test3() {
    fmt.Println("test3 方法................")
}

```
##  接口赋值 

接口赋值在 golang 语言中分为如下两种情况:

- 将对象实例赋值给接口。
- 将一个接口赋值给另一个接口。

## 类型查询

在Go语言中，还可以更加直截了当地询问接口指向的对象实例的类型，例如:

```go 
var v1 interface{} = ... 
switch v := v1.(type) {
    case int: // 现在v的类型是int 
    case string: // 现在v的类型是string 
    //...
}
```
----

参考资料：

1、[Golang 学习——interface 接口学习（一）](https://learnku.com/articles/44099)

2、[Golang 学习——interface 接口学习（二）](https://learnku.com/articles/44168)