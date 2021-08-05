---
layout: post
title: go build 时的错误分析
date: 2018-03-01   11:21:12
categories: 编程语言 go
tags:  编程语言 go
excerpt: go build 时的错误分析
---


go build 程序时，如果出现：

```sh
warning: building out-of-date packages:

runtime/pprof

testing

regexp/syntax

regexp

installing these packages with 'go test -i' will speed future tests.
```
 

那么就是说明下面的包已经有修改过了，但是没有重新install

如果有标准的包过期，使用go install -a -v std来进行更新

如果是自定义的包过期，重新调用go install