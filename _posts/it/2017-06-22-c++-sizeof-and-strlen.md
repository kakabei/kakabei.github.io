---
layout: post
title: c++ sizeof 和 strlen 的区别
date: 2017-06-22 16:12:15
categories: c++  
tags: c++  技术学习笔记
excerpt: strlen 是函数，sizeof 是c++运算符
---


1、 strlen 是头文件` <cstring>` 中的函数，sizeof 是 C++ 中的运算符。

2、 strlen 测量的是字符串的实际长度（其源代码如下），以 \0 结束。而 sizeof 测量的是字符数组的分配大小。

3、若字符数组 arr 作为函数的形参，sizeof(arr) 中 arr 被当作字符指针来处理，strlen(arr) 中 arr 依然是字符数组，从下述程序的运行结果中就可以看出。

4、strlen 本身是库函数，因此在程序运行过程中，计算长度；而 sizeof 在编译时计算长度；

5、sizeof 的参数可以是类型，也可以是变量；strlen 的参数必须是 char* 类型的变量。


```c++
size_t strlen(const char *str) {
    size_t length = 0;
    while (*str++)
        ++length;
    return length;
}

#include <iostream>
#include <cstring>

using namespace std;

void size_of(char arr[])
{
    // warning: 'sizeof' on array function parameter 'arr' will return size of 'char*' .
    cout << sizeof(arr) << endl; 
    cout << strlen(arr) << endl; 
}

int main()
{
    char arr[20] = "hello";
    size_of(arr); 
    cout << strlen(arr) << endl; // 5
    cout << sizeof(arr) << endl; // 20
    return 0;
}

/*
输出结果：
8
5
5
20
*/
```

sizeof(1==1) 在 C 和 C++ 中分别是什么结果？

```c++

// c
#include<stdio.h>

void main(){
    printf("%d\n", sizeof(1==1));
}

/*
运行结果：
4
*/

// c++ 

#include <iostream>
using namespace std;

int main() {
    cout << sizeof(1==1) << endl;
    return 0;
}

/*
1
*/
```

说明：

- C语言 : `sizeof（1 == 1） === sizeof（1）`按照整数处理，所以是4字节，这里也有可能是 8 字节（看操作系统）。
- C++ 因为有 bool 类型, `sizeof（1 == 1） == sizeof（true）` 按照 bool 类型处理，所以是 1 个字节。 
