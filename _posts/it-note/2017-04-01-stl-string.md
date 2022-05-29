---
layout: post
title: stl string 笔记
date: 2017-04-01 14:12:15
categories: stl
tags: stl note 
excerpt: stl string 杂记
---

1.string.h和cstring都不是string类的头文件，这两个头文件 主要地定义C语言铅笔字符趾操作的一些方法 ，譬如strlen()、strcpy()等。string.h是C语言的头文件格式，而cstring是C++风格的头文件，但是和<string.h>是一样的，它的目的是为了和C语言兼容。

2.string的一道面试题。很编写一个类string的7个函数。（后补）

3.C++字符串和C字符串的转换

C++提供了的由C++字符 串转换成对应的C字符串的方法是使用data()、c_str()和copy()来实现。

其中，data()以字符数据 的形式返回 字符串内容，但并不添加 '\0'；c_str()返回一以'\0'结尾的字符数组；而copy()则把字符 串的内容复制或写入既有的c_string或字符数组内，需要注意是，c++字符串并以不'\0'结尾。

c_str()语句可以生成一个 const char *  指针，江脂向空字符 的数组。这个数组的数据 是临时的， 当有一个改变这数据的成员函数被调用后，其中的数据就会失效。因此要么现用现转换，要么把它的数据复制到用户自己可以管理 的内在中后再转换。

4.string和int的类型转换

（1） int转string的方法。

可以用snprintf()。
```c
int snprintf(char *str, size_t size, const char *format, ...)
```
（2）string 转int的方法 

可以用 strtol， strtoll，strtoul，或strtoull函数。

5.string 其他常用成员函数

```c
int capacity() const ;             // 返回当前容量（即string 中不必增加内在即可存的元素个数）
int max_size()const;               // 返回string对象中可存放的最大字符串的长度
int size() const ;                 // 返回当前字符串的大小 
int length() const;                // 返回当前字符串的长度
int empty() const;                 // 当前字符串是否为空
void resize(int len, char c) ;      // 把字符串当前大小为为len，并用字符c填充不足的部分。
```

6、 string的size()和length()的结果是一样的。

length是因为沿用C语言的习惯而保留下来的，string类最初只有length，引入STL之后，为了兼容又加入了size，它是作为STL容器的属性存在的，便于符合STL的接口规则，以便用于STL的算法。string类的size()/length()方法返回的是字节数，不管是否有汉字。


 

