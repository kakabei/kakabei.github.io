---
layout: post
title: c++ 14 新特性
date: 2021-01-03 16:12:15
categories: 编程语言  
tags: c++ c 技术笔记
excerpt: C++14 新特性主要体现在三个领域：Lambda 函数、constexpr 和类型推导
---


C++14 这一继 C++11 之后的新的 C++ 标准。是一个小版本。

C++14 新特性主要体现在三个领域：Lambda 函数、constexpr 和类型推导。

![](/assets/programming-language/c-plus-plus-14-2022-11-21_11-34-12.png)

# Lambda 函数

C++14 的 Lambda 函数可以使用推导类型声明。如：

```c++ 14

auto lambda = [](auto a, auto b) {return a + b;};

```

而 C++ 11 的 Lambda 函数使用的是具体的类型声明，如

```c++

auto lambda = [](int a, int b) {return a + b;};

```

# constexpr

c++11 关于constexpr 的特性可以看： [[c++11  constexpr]]  

在 C++11 中，使用 constexpr 声明的函数可以在编译时执行，生成一个值，用在需要常量表达式的地方，比如作为初始化模板的整形参数。C++11 的 constexpr 函数只能包含一个表达式，C++14 放松了这些限制，支持诸如 if 和 switch 等条件语句，支持循环，其中包括基于区间（range）的 for 循环。

如：

```c++
constexpr int factorial(int n) { // C++11中不可，C++14中可以
    int ret = 0;
    for (int i = 0; i < n; ++i) {
        ret += i;
    }
    return ret;
}
```


#  类型推导

C++11 仅支持 Lambda 函数的类型推导，C++14 对其加以扩展，支持所有函数的返回类型推导：

```c++
#include <iostream>

using namespace std;

auto func(int a, int b) {
    return a + b;
}

int main() {
    cout << func(4,5) << endl;
    return 0;
}
```

也可以模板中使用 `auto`  如：

```c++
#include <iostream>
using namespace std;

template<typename T> auto func(T a, T b) { return a+b; }

int main() {
    cout << func(4, 5) << endl;
    cout << func(3.4, 4.5) << endl;
    return 0;
}
```

`auto`  做为函数的返回值要注意的地方：

1、函数如果有多个return，返回的类型必须是相同的，否则编译失败。

2、如果return语句返回初始化列表，返回值类型推导也会失败。

3、如果函数是虚函数，不能使用返回值类型推导。

4、返回类型推导可以用在前向声明中，但是在使用它们之前，翻译单元中必须能够得到函数定义。

5、返回类型推导可以用在递归函数中，但是递归调用必须以至少一个返回语句作为先导，以便编译器推导出返回类型。


#  二进制字面量与整形字面量分隔符

C++14引入了二进制字面量，也引入了分隔符

```c++
int a = 0b0001'0011'1010;
double b = 3.14'1234'1234'1234;
```

# deprecated 标记

C++14 中增加了deprecated标记，修饰类、变、函数等，当程序中使用到了被其修饰的代码时，编译时被产生警告，用户提示开发者该标记修饰的内容将来可能会被丢弃，尽量不要使用。

```c++
struct [[deprecated]] A { };

int main() {
    A a;
    return 0;
}
	
```

# std::make_unique

C++11中有std::make_shared，却没有std::make_unique，在C++14 添加上了。

```c++
struct A {};
std::unique_ptr<A> ptr = std::make_unique<A>();
```


# 其他

1、`std::shared_timed_mutex` 和 `std::shared_lock` 来实现读写锁，保证多个线程可以同时读，但是写线程必须独立运行，写操作不可以同时和读操作一起进行。

2、`std::integer_sequence`

3、`std::exchange` 和 `stl` 的 `std::swap` 类似。不同的是 `std::swap` 是交换，而 `std::exchange` 是把后面的值赋给前面的，同时把旧值返回。

4、`std::quoted` 用于给字符串添加双引号。



----
reference :

1、[C++14 新特性总结](https://www.infoq.cn/article/2014/09/cpp14-here-features)

2、[C++14新特性的所有知识点全在这儿啦！](https://segmentfault.com/a/1190000023441427)

3、 [Implementation C++14 make_integer_sequence](https://stackoverflow.com/questions/17424477/implementation-c14-make-integer-sequence)
