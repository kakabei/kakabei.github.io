---
layout: post
title: c++11 初始化列表
date: 2017-05-02 15:12:15
categories: 编程语言 
tags: c++  技术阅读笔记
excerpt: c++11 初始化列表
---


initializer_list是C++11的的一个新特性。

c++11之前我们对vector的初始化是这样的：

```c++
std::vector vec;
vec.push_back(1);
vec.push_back(2);
vec.push_back(3);

```

C++11之后，可以用花括号直接经初始化。我们可以这样:

```c++
std::vector vec = { 1, 2, 3};
```

这就是所谓的initializer list。

所以有initializer_list

C++11允许构造函数和其他函数把初始化列表当做参数。

```c++
#include <iostream>
#include <vector>

class MyHome
{
public:
	MyHome(const std::initializer_list<int> &vec)
	 {
		for (auto item : vec)
		{
            m_Vec.push_back(item);
        }
    }

	void print()
	{
		for (auto item : m_Vec)
		{
        	std::cout << item << " ";
    	}
    }
private:
    std::vector<int> m_Vec;
};

int main()
{
    MyHome m = { 1, 2, 3};
    m.print();  // 1 2 3

    return 0;
}
```

###### 用 initializer_list 的好处：
---
1.initializer_list不能修改，更符合参数的特点。

vector有push_back函数，也就是说vector可以在函数里面修改，所以必然vector必须在heap上分配空间来存储数据。
而initializer_list只有begin和end函数，函数内并不能修改它，所以编译器有机会在stack上存储initializer_list的数据来提高性能。
 
2.initializer_list是指针语义

vector是值语义，也就是说拷贝一个vector，那里面的元素也会被拷贝一次。而initializer_list是指针语义，里面的元素并不会被拷贝。是指向了同一个空间。


3.初始化列表还可以用于函数返回的情况。 返回一个初始化列表，通常会导致构造一个临时变量，比如：
 
 ```c++
 vector< int> Func() { return {1, 3}; }
```

4.防止类型收窄

用法很简单
```c++
const int x = 1024; 
const int y = 10; 
char a = x;             // 收窄， 但可以通过编译 

char c = {x};           // 收窄， 无法通过编译 
char d = {y};           // 可以通过编译

```



---
\--《深入理解C++11：C++11新特性解析与应用》
