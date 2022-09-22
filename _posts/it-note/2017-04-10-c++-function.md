---
layout: post
title: c++ 关键字库函数知识点
date: 2017-04-10 16:12:15
categories: 编程语言 
tags: c++  
excerpt: c++ 关键字库函数知识点 
---


## 1 sizeof 和 strlen 的区别

1、 strlen 是头文件` <cstring>` 中的函数，sizeof 是 C++ 中的运算符。

2、 strlen 测量的是字符串的实际长度（其源代码如下），以 \0 结束。而 sizeof 测量的是字符数组的分配大小。

3、若字符数组 arr 作为函数的形参，sizeof(arr) 中 arr 被当作字符指针来处理，strlen(arr) 中 arr 依然是字符数组，从下述程序的运行结果中就可以看出。

4、strlen 本身是库函数，因此在程序运行过程中，计算长度；而 sizeof 在编译时，计算长度；

5、 sizeof 的参数可以是类型，也可以是变量；strlen 的参数必须是 char* 类型的变量。


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
    char arr[10] = "hello";
    
    return 0;
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

C语言
sizeof（1 == 1） === sizeof（1）按照整数处理，所以是4字节，这里也有可能是8字节（看操作系统）
C++
因为有bool 类型
sizeof（1 == 1） == sizeof（true） 按照bool类型处理，所以是1个字节。 




## 2 lambda 匿名函数

`lambda` 表达式是一个可调度的代码单元，可以视为一个未命名的内部函数
`lambda` 函数是一个函数对象，编译器在编译时会生成一个 `lambda` 对象的类，然后再生成一个该命令未命名的对象
lambda 的形式如下：

```
[捕获列表] (参数列表) -> 返回类型 { 函数部分 }
[capture list] (parameter list) -> return type { function body }
```

capture list 捕获列表是 `lambda` 函数所定义的函数的局部变量列表， 通常为空
一个 lambda 只有在其捕获列表中捕获一个所在函数中的局部变量，才能在函数体中使用该变量。

捕获列表只用于局部非 static 变量。 lambda 可以直接使用局部变量 static 变量 和在它所在函数之外的声明的名字。

捕获列表的变量可以分为 值 或 引用传递。

值传递： lambda 捕获的变量在 lambda 函数 创建 就发生了拷贝而非调用时。

隐式捕获：
编译器可以根据 lambda 中的代码推导使用的变量，为指示编译器推断捕获列表，应该在捕获列表中写一个 ``&`` 或 ``=``

`&` 告知编译器采用引用传递方式
`=` 告知编译器采用值传递方式

 当混合使用时，捕获列表第一个参数必须是 `&` 或 `= `且显示捕获的变量必须和隐式捕获使用不同的传递方式

`pameter list`
参数列表和普通函数类似，但是 lambda 不能有 默认参数【lambda 实参和形参数目一定相等】

`return type`

与普通函数不同的是: lambda 必须使用位尾置返回 来指定返回类型。

如果忽略返回类型，lambda 表达式会根据函数体中的代码推断出返回类型
若函数体只有一个 return 语句， 则返回类型从返回表达式的类型推断而来，否则，若未指定返回类型，返回类型为 void

Note: 如果 lambda 的函数体包含任意单一 return 之外的内容， 且未指定返回类型，则返回 void
当需要为 lambda 定义返回类型时，必须使用尾置返回类型

`function body`
与常规函数类似

## 2  explicit 的作用

用来声明类构造函数是显示调用的，而非隐式调用，可以阻止调用构造函数时进行隐式转换。

```c++
#include <iostream>
#include <cstring>
using namespace std;

class A
{
public:
    int var;
    explicit A(int tmp)
    {
        var = tmp;
        cout << var << endl;
    }
};
int main()
{
    A ex(100);
    A ex1 = 10; // error: conversion from 'int' to non-scalar type 'A' requested
    return 0;
}


```

说明，发现隐性调用： 

```c++ 
A ex1 = 10;
///

A ex2(10); 
A ex1 = ex2; 


```

## 3  static 的作用和说明

1、  static 定义变量的位置在静态变量区，超过其作用域该变量并不被释放，而是在函数结束时释放

2、 static 修饰的变量只会被初始化一次

3、  static 修饰类：

	- static 修饰的成员变量要在类外初始化，属于类，为所有类对象共享，static 修饰的变量不占类的空间
	- static 修饰的函数，静态成员函数， 属于类，为类的所有对象共享， 不能访问类的非静态成员，和外部函数， 没有this指针，因此只能访问静态成员(静态成员变量和静态函数)

4、 static 静态成员变量：

	- 静态成员变量是在类内进行声明，在类外进行定义和初始化，在类外进行定义和初始化的时候不要出现 static 关键字和private、public、protected 访问规则。

	- 静态成员变量相当于类域中的全局变量，被类的所有对象所共享，包括派生类的对象。

	- 静态成员变量可以作为成员函数的参数，而普通成员变量不可以。

```c++
#include <iostream>
using namespace std;

class A
{
public:
    static int s_var;
    int var;
    void fun1(int i = s_var); // 正确，静态成员变量可以作为成员函数的参数
    void fun2(int i = var);   //  error: invalid use of non-static data member 'A::var'
};
int main()
{
    return 0;
}

```


5、 静态数据成员的类型可以是所属类的类型，而普通数据成员的类型只能是该类类型的指针或引用。

```c++
#include <iostream>
using namespace std;

class A
{
public:
    static A s_var; // 正确，静态数据成员
    A var;          // error: field 'var' has incomplete type 'A'
    A *p;           // 正确，指针
    A &var1;        // 正确，引用
};

int main()
{
    return 0;
}

```

6、static 静态成员函数：

	- 静态成员函数不能调用非静态成员变量或者非静态成员函数，因为静态成员函数没有 this 指针。静态成员函数做为类作用域的全局函数。

	- 静态成员函数不能声明成虚函数（virtual）、const 函数和 volatile 函数。

7、 static 全局变量和普通全局变量的异同

相同点：

	存储方式：普通全局变量和 static 全局变量都是静态存储方式。

不同点：

	作用域：普通全局变量的作用域是整个源程序，当一个源程序由多个源文件组成时，普通全局变量在各个源文件中都是有效的；静态全局变量则限制了其作用域，即只在定义该变量的源文件内有效，在同一源程序的其它源文件中不能使用它。由于静态全局变量的作用域限于一个源文件内，只能为该源文件内的函数公用，因此可以避免在其他源文件中引起错误。

	初始化：静态全局变量只初始化一次，防止在其他文件中使用。


8、 返回函数中静态变量的地址会发生什么？

```c++ 
#include <iostream>
using namespace std;

int * fun(int tmp){
    static int var = 10;
    var *= tmp;
    return &var;
}

int main() {
    cout << *fun(5) << endl;
    return 0;
}

/*
运行结果：
50
*/

```
说明：上述代码中在函数 fun 中定义了静态局部变量 var，使得离开该函数的作用域后，该变量不会销毁，返回到主函数中，该变量依然存在，从而使程序得到正确的运行结果。但是，该静态局部变量直到程序运行结束后才销毁，浪费内存空间。

全局变量（包括静态全局变量）是最先构造的，早于main函数，当然，析构函数也是执行的最晚，晚于main函数。

静态局部变量是要等到执行该声明定义的表达式后，才开始执行构造的。当然，析构函数也是早于全局变量的。



## 4 const 作用和及用法

1、 const 修饰成员变量，定义成 const 常量，相较于宏常量，可进行类型检查，节省内存空间，提高了效率。

2、 const 修饰函数参数，使得传递过来的函数参数的值不能改变。

3、 const 修饰成员函数，使得成员函数不能修改任何类型的成员变量（mutable 修饰的变量除外），也不能调用非 const 成员函数，因为非 const 成员函数可能会修改成员变量。

4、define 和 const 的区别

	编译阶段：define 是在编译预处理阶段进行替换，const 是在编译阶段确定其值。
		
	安全性：define 定义的宏常量没有数据类型，只是进行简单的替换，不会进行类型安全的检查；const 定义的常量是有类型的，是要进行判断的，可以避免一些低级的错误。

	内存占用：define 定义的宏常量，在程序中使用多少次就会进行多少次替换，内存中有多个备份，占用的是代码段的空间；const 定义的常量占用静态存储区的空间，程序运行过程中只有一份。

	调试：define 定义的宏常量不能调试，因为在预编译阶段就已经进行替换了；const 定义的常量可以进行调试。

5、 const 的优点：

	- 有数据类型，在定义式可进行安全性检查。
	- 可调式。
	- 占用较少的空间。

6、define 和 typedef 的区别：

	- 原理：#define 作为预处理指令，在编译预处理时进行替换操作，不作正确性检查，只有在编译已被展开的源程序时才会发现可能的错误并报错。typedef 是关键字，在编译时处理，有类型检查功能，用来给一个已经存在的类型一个别名，但不能在一个函数定义里面使用 typedef 。

	- 功能：typedef 用来定义类型的别名，方便使用。#define 不仅可以为类型取别名，还可以定义常量、变量、编译开关等。

	- 作用域：#define 没有作用域的限制，只要是之前预定义过的宏，在以后的程序中都可以使用，而 typedef 有自己的作用域。

- 指针的操作  ?

7、用宏实现比较大小，以及两个数中的最小值

```c++

#include <iostream>
#define MAX(X, Y) ((X)>(Y)?(X):(Y))
#define MIN(X, Y) ((X)<(Y)?(X):(Y))
using namespace std;


int main ()
{
    int var1 = 10, var2 = 100;
    cout << MAX(var1, var2) << endl;
    cout << MIN(var1, var2) << endl;
    return 0;
}
/*
程序运行结果：
100
10
*/

```

## 5 inline 作用及使用方法

1、 inline 是一个关键字，可以用于定义内联函数。内联函数，像普通函数一样被调用，但是在调用时并不通过函数调用的机制而是直接在调用点处展开，这样可以大大减少由函数调用带来的开销，从而提高程序的运行效率。

2、 类内定义成员函数默认是内联函数
在类内定义成员函数，可以不用在函数头部加 inline 关键字，因为编译器会自动将类内定义的函数（构造函数、析构函数、普通成员函数等）声明为内联函数。 

3、类外定义成员函数，若想定义为内联函数，需用关键字声明
当在类内声明函数，在类外定义函数时，如果想将该函数定义为内联函数，则可以在类内声明时不加 inline 关键字，而在类外定义函数时加上 inline 关键字。

4、另外，可以在声明函数和定义函数的同时加上 inline；也可以只在函数声明时加 inline，而定义函数时不加 inline。只要确保在调用该函数之前把 inline 的信息告知编译器即可。

```c++
#include <iostream>
using namespace std;

class A{
public:
    int var;
    A(int tmp){ 
      var = tmp;
    }    
    void fun(){ 
        cout << var << endl;
    }
};

int main()
{    
    return 0;
}


```

5 、inline 作用及使用方法

	- 内联函数不是在调用时发生控制转移关系，而是在编译阶段将函数体嵌入到每一个调用该函数的语句块中，编译器会将程序中出现内联函数的调用表达式用内联函数的函数体来替换。

	- 普通函数是将程序执行转移到被调用函数所存放的内存地址，当函数执行完后，返回到执行此函数前的地方。转移操作需要保护现场，被调函数执行完后，再恢复现场，该过程需要较大的资源开销。


6、 宏定义（define）和内联函数（inline）的区别

	- 内联函数是在编译时展开，而宏在编译预处理时展开；在编译的时候，内联函数直接被嵌入到目标代码中去，而宏只是一个简单的文本替换。

		-内联函数是真正的函数，和普通函数调用的方法一样，在调用点处直接展开，避免了函数的参数压栈操作，减少了调用的开销。而宏定义编写较为复杂，常需要增加一些括号来避免歧义。

	- 宏定义只进行文本替换，不会对参数的类型、语句能否正常编译等进行检查。而内联函数是真正的函数，会对参数的类型、函数体内的语句编写是否正确等进行检查。

## 6 new 和 malloc

1、new 和 malloc 如何判断是否申请到内存？

	- malloc ：成功申请到内存，返回指向该内存的指针；分配失败，返回 NULL 指针。

	- new ：内存分配成功，返回该对象类型的指针；分配失败，抛出 bad_alloc 异常。

可以使用 `std::nothrow` 让 `new` 在申请内存失败时也同 `malloc`  一样返回 `NULL` 指针，而不是抛出 `std::bad_alloc`   异常。
```c++
A *a = new (std::nothrow) A();
if (a == nullptr) {
    // add logs here
    return false;
}

```

2、 delete 实现原理？delete 和 delete[] 的区别？

delete 的实现原理：首先执行该对象所属类的析构函数；进而通过调用 operator delete 的标准库函数来释放所占的内存空间。

delete 和 `delete []` 的区别：
	
	- delete 用来释放单个对象所占的空间，只会调用一次析构函数；
	
	- `delete []` 用来释放数组空间，会对数组中的每个成员都调用一次析构函数。
		
		对于像 `int` / `char` /`long/int*` 等等简单数据类型，由于对象没有析构函数，所以用 delete 和 `delete []` 是一样的！都不会造成内存泄露！ 但通常为了规范起见，`new []`都配套使用 `delete []`。

但是如果是C++自定义对象数组就不同了！由于delete p只调用了一次析构函数，剩余的对象不会调用析构函数，所以剩余对象中如果有申请了新的内存或者其他系统资源，那么这部分内存和资源就无法被释放掉了，因此会造成内存泄露或者更严重的问题。



3、  new 和 malloc 的区别，delete 和 free 的区别

	- malloc、free 是库函数，而new、delete 是关键字。
	
	- new 申请空间时，无需指定分配空间的大小，编译器会根据类型自行计算；malloc 在申请空间时，需要确定所申请空间的大小。
	
	- new 申请空间时，返回的类型是对象的指针类型，无需强制类型转换，是类型安全的操作符；malloc 申请空间时，返回的是 void* 类型，需要进行强制类型的转换，转换为对象类型的指针。
	
	- new 分配失败时，会抛出 bad_alloc 异常，malloc 分配失败时返回空指针。

	- 对于自定义的类型，new 首先调用 operator new() 函数申请空间（底层通过 malloc 实现），然后调用构造函数进行初始化，最后返回自定义类型的指针；
	
	- delete 首先调用析构函数，然后调用 operator delete() 释放空间（底层通过 free 实现）。malloc、free 无法进行自定义类型的对象的构造和析构。

	- new 操作符从自由存储区上为对象动态分配内存，而 malloc 函数从堆上动态分配内存。（自由存储区不等于堆）

这里的自由存储区应该怎么理解？ 

4、  malloc 的底层实现？

malloc 的原理:

当开辟的空间小于 128K 时，调用 brk() 函数，通过移动 _enddata 来实现；当开辟空间大于 128K 时，调用 mmap() 函数，通过在虚拟地址空间中开辟一块内存空间来实现。

malloc 的底层实现：

brk() 函数实现原理：向高地址的方向移动指向数据段的高地址的指针 _enddata。

mmap 内存映射原理：

进程启动映射过程，并在虚拟地址空间中为映射创建虚拟映射区域；

调用内核空间的系统调用函数 mmap()，实现文件物理地址和进程虚拟地址的一一映射关系；进程发起对这片映射空间的访问，引发缺页异常，实现文件内容到物理内存（主存）的拷贝。


内存分配的原理

从操作系统角度来看，进程分配内存有两种方式，分别由两个系统调用完成：brk和mmap（不考虑共享内存）。

1、brk是将数据段(.data)的最高地址指针_edata往高地址推；

2、mmap是在进程的虚拟地址空间中（堆和栈中间，称为文件映射区域的地方）找一块空闲的虚拟内存。

这两种方式分配的都是虚拟内存，没有分配物理内存。在第一次访问已分配的虚拟地址空间的时候，发生缺页中断，操作系统负责分配物理内存，然后建立虚拟内存和物理内存之间的映射关系。

在标准C库中，提供了malloc/free函数分配释放内存，这两个函数底层是由brk，mmap，munmap这些系统调用实现的。

## 7  struct 

1 、C 和 C++ struct 的区别？

在 C 语言中 struct 是用户自定义数据类型；在 C++ 中 struct 是抽象数据类型，支持成员函数的定义。

C 语言中 struct 没有访问权限的设置，是一些变量的集合体，不能定义成员函数；C++ 中 struct 可以和类一样，有访问权限，并可以定义成员函数。

C 语言中 struct 定义的自定义数据类型，在定义该类型的变量时，需要加上 struct 关键字，例如：struct A var;，定义 A 类型的变量；而 C++ 中，不用加该关键字，例如：A var;

在c++中struct和class的唯一区别是成员的默认访问权限。struct默认是public，而class默认是private。

C++ 是在 C 语言的基础上发展起来的，为了与 C 语言兼容，C++ 中保留了 `struct`。

2、 struct 和 class 的区别

struct 和 class 都可以自定义数据类型，也支持继承操作。

struct 中默认的访问级别是 public，默认的继承级别也是 public；class 中默认的访问级别是 private，默认的继承级别也是 private。
	
当 class 继承 struct 或者 struct 继承 class 时，默认的继承级别取决于 class 或 struct 本身， class（private 继承），struct（public 继承），即取决于派生类的默认继承级别。


3、 struct 和 union 的区别： 

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

u22的size为什么等于12？

结果是union内 max(最大字段的大小) + n(填充大小) 应该等于 union中 max_basic(最大基本类型大小)的倍数
res = max+n;
例子中, 最大字段是 char c[10] 占10字节， 最大基本类型是 int i 占4字节， 按上述规则，填充2字节，答案即为12（12= 4x3）

4、 如何判断结构体是否相等

需要重载操作符 == 判断两个结构体是否相等，不能用函数 memcmp 来判断两个结构体是否相等，因为 memcmp 函数是逐个字节进行比较的，而结构体存在内存空间中保存时存在字节对齐，字节对齐时补的字节内容是随机的，会产生垃圾值，所以无法比较。

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


## 8  volatile 作用

volatile变量的作用和使用场景如下：

在多线程编程中，多个线程访问同一个变量，而且有可能修改变量，如果不加volatile，编译器有可能做优化，在寄存器中保存变量，CPU直接访问寄存器，从而优化变量访问速度，加上volatile修饰，CPU会直接访问内存，不会做寄存器的优化。这一条称为保证内存的可见性。

类似的情况在中断服务程序中访问变量，并行设备的硬件寄存器变量都是一个道理。

另一个作用是禁止指令重排，也是针对编译器说的，保证缓存一致性。

volatile保证了程序的正确性，损失了编译器的优化。

能干什么：

告诉编译器不要在此内存上做任何优化。如果对内存有只写未读的等非常规操作，如
```c++
`x=10; x=20;`
```

编译器会优化为

```c++
`x=20;`
```


## 9  extern C 的作用

C++ 和 C语言编译函数签名方式不一样， extern关键字可以让两者保持统一，这样才能找到对应的函数。

c和c++对同一个函数经过编译后生成的函数名是不同的，由于C++支持函数重载，因此编译器编译函数的过程中会将函数的参数类型也加到编译后的代码中，而不仅仅是函数名；

而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，一般只包括函数名。如果在c++中调用一个使用c语言编写的模块中的某个函数，那么c++是根据c++的名称修饰方式来查找并链接这个函数，那么就会发生链接错误。


## 10 memmove 函数的底层原理

memmove  和 memcpy 都是c 语言的库函数， 定义在 `string.h` 中， 他们的作用一样，惟一的区别是：当内部发生重叠后，memmove 可以保证数据正确性。
```c++

void *memcpy(void * dst, cosnt void * src, size_t n)
{
	char * tmp = (char *)dst; 
	char * s_str = (char *)src; 

	while (n--){
		*tmp++ = *s_str++; 
	}
	return dst; 
}


void *memmove(void *dst, const void *src, size_t size)
{
    char *psrc;
    char *pdst;

    if (NULL == dst || NULL == src)
    {
        return NULL;
    }

	// 这个条件不是特别能理解？？？
    if ((src < dst) && (char *)src + size > (char *)dst) // 出现地址重叠的情况，自后向前拷贝
    {
        psrc = (char *)src + size - 1;
        pdst = (char *)dst + size - 1;
        while (size--)
        {
            *pdst-- = *psrc--;
        }
    }
    else
    {
        psrc = (char *)src;
        pdst = (char *)dst;
        while (size--)
        {
            *pdst++ = *psrc++;
        }
    }

    return dst;
}

```

strcpy 函数有什么缺陷？

`trcpy` 函数不检查目的缓冲区的大小边界，而是将源字符串逐一的全部赋值给目的字符串地址起始的一块连续的内存空间，同时加上字符串终止符，会导致其他变量被覆盖。

```c++
char* strcpy(char* des,const char* source)
{
    char* r=des;
    assert((des != NULL) && (source != NULL));
    while((*r++ = *source++)!='\0');
    return des;
}

```

## 11 auto 类型推导

auto变量的规则是"做函数模板需要做的事情"

```c++
auto my_new_variable = its_initial_value;
```

就像普通的赋值操作一样,my_new_variable的基本类型和值与its_initial_value一样,但是its_initial_value的第二属性(顶层const,valatile, &/&&)不一定相同.例如its_initial_value是const 并不意味着my_new_variable是const.因此,my_new_variable和its_initial_value只是在基本类型上相同,但不是完全相同的变量.

---
from leetcode [C++ 面试突破 https://leetcode.cn/leetbook/detail/cmian-shi-tu-po/](https://leetcode.cn/leetbook/detail/cmian-shi-tu-po/)