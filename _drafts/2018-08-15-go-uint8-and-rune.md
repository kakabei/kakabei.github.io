---
layout: post
title: golang 字符类型 byte 和 rune
date: 2018-08-11  16:12:15
categories: golang  
tags:  读书 golang
excerpt: golang 的字符有两种 uint8 和 rune
---


golang 的字符有两种：
-  `uint8` 类型，或叫 `byte` 型。代表了 `ASCII` 码的一个字符。
-  `rune` 类型，代表一个UTF-8字符，当需要处理复合字符时，则需要用到 rune 类型。 rune 类型等价 `int32` 类型。

传统 `ASCII` 编码只占用 1 个字节，所以可以用 `byte` 类型来表示， 如 

```go
var ch byte='A' 		// 字符使用单引号括起来。
var ch byte = 65 		// 十进制表示
var ch byte = '\x41'    //（\x 总是紧跟着长度为 2 的 16 进制数）
```

在书写 `Unicode` 字符时，需要在 16 进制数之前加上前缀 `\u` 或者 `\U`。因为 `Unicode` 至少占用 2 个字节，所以我们使用 `int16` 或者 `int` 类型来表示。如果需要使用到 4 字节，则使用`\u`前缀，如果需要使用到 8 个字节，则使用`\U`前缀。

```go
var ch1 int = '\u0041'
var ch2 int = '\u03B2'
var ch3 int = '\U00101234'
```

Unicode 包中内置了一些用于测试字符的函数，这些函数的返回值都是一个布尔值，如下所示（其中 ch 代表字符）：

- 判断是否为字母：`unicode.IsLetter(ch)`
- 判断是否为数字：`unicode.IsDigit(ch)`
- 判断是否为空白符号：`unicode.IsSpace(ch)`


****