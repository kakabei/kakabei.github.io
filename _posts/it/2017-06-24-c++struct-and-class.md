---
layout: post
title: c++ struct 和 class 的区别
date: 2017-06-24 16:12:15
categories: c++  
tags: c++  技术学习笔记
excerpt: 当字符串内部发生重叠后，memmove 可以保证数据正确性
---


 

### 1、C 和 C++ `struct` 的区别？

在 C 语言中 `struct` 是用户自定义数据类型；在 C++ 中 `struct` 是抽象数据类型，支持成员函数的定义。

C 语言中 `struct` 没有访问权限的设置，是一些变量的集合体，不能定义成员函数；C++ 中 `struct` 可以和类一样，有访问权限，并可以定义成员函数。

C 语言中 `struct` 定义的自定义数据类型，在定义该类型的变量时，需要加上 `struct` 关键字，例如：`struct A var;`，定义 A 类型的变量；而 C++ 中，不用加该关键字，例如：A var;

在c++中 `struct` 和 `class` 的唯一区别是成员的默认访问权限。`struct` 默认是 public，而 `class` 默认是 private。

C++ 是在 C 语言的基础上发展起来的，为了与 C 语言兼容，C++ 中保留了 `struct`。

### 2、 `struct` 和 `class` 的区别?

`struct` 和 `class` 都可以自定义数据类型，也支持继承操作。

`struct` 中默认的访问级别是 public，默认的继承级别也是 public；`class` 中默认的访问级别是 private，默认的继承级别也是 private。
	
当 `class` 继承 `struct` 或者 `struct` 继承 `class` 时，默认的继承级别取决于 `class` 或 `struct` 本身， `class`（private 继承），`struct`（public 继承），即取决于派生类的默认继承级别。

### 3、 `struct` 和 `union` 的区别?

`union` 是联合体，`struct` 是结构体。

联合体和结构体都是由若干个数据类型不同的数据成员组成。使用时，联合体只有一个有效的成员；而结构体所有的成员都有效。

对联合体的不同成员赋值，将会对覆盖其他成员的值，而对于结构体的对不同成员赋值时，相互不影响。

联合体的大小为其内部所有变量的最大值，按照最大类型的倍数进行分配大小；结构体分配内存的大小遵循内存对齐原则。

```c++
#include <iostream>
using namespace std;

typedef union
{
    char c[10];
    char cc1; // char 1 字节，按该类型的倍数分配大小
} u11;

typedef union
{
    char c[10];
    int i; // int 4 字节，按该类型的倍数分配大小
} u22;

typedef union
{
    char c[10];
    double d; // double 8 字节，按该类型的倍数分配大小
} u33;

typedef struct s1
{
    char c;   // 1 字节
    double d; // 1（char）+ 7（内存对齐）+ 8（double）= 16 字节
} s11;

typedef struct s2
{
    char c;   // 1 字节
    char cc;  // 1（char）+ 1（char）= 2 字节
    double d; // 2 + 6（内存对齐）+ 8（double）= 16 字节
} s22;

typedef struct s3
{
    char c;   // 1 字节
    double d; // 1（char）+ 7（内存对齐）+ 8（double）= 16 字节
    char cc;  // 16 + 1（char）+ 7（内存对齐）= 24 字节
} s33;

int main()
{
    cout << sizeof(u11) << endl; // 10
    cout << sizeof(u22) << endl; // 12
    cout << sizeof(u33) << endl; // 16
    cout << sizeof(s11) << endl; // 16
    cout << sizeof(s22) << endl; // 16
    cout << sizeof(s33) << endl; // 24

    cout << sizeof(int) << endl;    // 4
    cout << sizeof(double) << endl; // 8
    return 0;
}

```

u22 的 size 为什么等于12？

结果是 `union` 内 max(最大字段的大小) + n(填充大小) 应该等于 `union` 中 max_basic(最大基本类型大小)的倍数
res = max+n;
例子中, 最大字段是 `char c[10]` 占10字节， 最大基本类型是 `int i` 占4字节， 按上述规则，填充2字节，答案即为 12（12= 4x3）

4、如何判断结构体是否相等

需要重载操作符 `==` 判断两个结构体是否相等，不能用函数 `memcmp` 来判断两个结构体是否相等，因为 `memcmp` 函数是逐个字节进行比较的，而结构体存在内存空间中保存时存在字节对齐，字节对齐时补的字节内容是随机的，会产生垃圾值，所以无法比较。

```c++
#include <iostream>

using namespace std;

struct A
{
    char c;
    int val;
    A(char c_tmp, int tmp) : c(c_tmp), val(tmp) {}

    friend bool operator==(const A &tmp1, const A &tmp2); //  友元运算符重载函数
};

bool operator==(const A &tmp1, const A &tmp2)
{
    return (tmp1.c == tmp2.c && tmp1.val == tmp2.val);
}

int main()
{
    A ex1('a', 90), ex2('b', 80);
    if (ex1 == ex2)
        cout << "ex1 == ex2" << endl;
    else
        cout << "ex1 != ex2" << endl; // 输出
    return 0;
}
```
