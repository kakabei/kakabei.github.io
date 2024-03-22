---
layout: post
title: golang 笔记 链接符号
date: 2018-03-07 07:21:12
categories:  golang 
tags: golang
excerpt: golang 链接符号关心的是如何将语言文法使用的符号转化为链接期使用的符号
---


链接符号关心的是如何将语言文法使用的符号转化为链接期使用的符号，在常规情况下，链接期使用的符号对我们不可见。

C++ 支持函数重载，故此语言的“链接符号”构成极其复杂，需要包括

- 命令空间 `Namespace`
- 类 `ClassType`
- 方法 `Method`
- 参数 `ArgType1, ArgType2,...`

不同编译器厂商生成“链接符号” 的规则并不统一，生成的二进制模块(.o 或 .so)是不兼容的。因此多数情况下，C++ 语言的模块间交互使用 C 的机制， 而不是自身的机制。

Go 语言中，一般化的函数原型如下：   

```go 
package Package
func Method(arg1 ArgType1, arg2 ArgType2, ...) (ret1 RetType1, ret2 RetType2, ...)
func (v ClassType) Method(arg1 ArgType1, arg2 ArgType2, ...) (ret1 RetType1, ret2 RetType2, ...)
func (this *ClassType) Method(arg1 ArgType1, arg2 ArgType2, ...) (ret1 RetType1, ret2 RetType2, ...) 
// 这种可以认为是上一种情况的特例
```

由于 Go 语言并无重载，故此语言的“链接符号”由如下信息构成:

- Package Package 名可以是多层，例如 `A/B/C`。
- ClassType  很特别的是，Go 语言中 `ClassType` 可以是指针，也可以不是。
- Method 

其“链接符号”的组成规则如下:

- `Package.Method`
- `Package.ClassType·Method`

举个例子,假设在 `qbox.us/mockfs` 模块中。

```go 
func New(cfg Config) *MockFS
func (fs *MockFS) Mkdir(dir string) (code int, err error) 
func (fs MockFS) Foo(bar Bar)
```

它们的链接符号分别为:

```text
qbox.us/mockfs.New
qbox.us/mockfs.*MockFS·Mkdir
qbox.us/mockfs.MockFS·Foo
```

