---
layout: post
title: c++11 右值引用
date: 2017-07-01 16:12:15
categories: c++  
tags: c++ 技术笔记
excerpt: 左值可以取地址、位于等号左边；而右值没法取地址，位于等号右边
---

from : [# 一文读懂C++右值引用和std::move](https://zhuanlan.zhihu.com/p/335994370)

# 一、什么是左值、右值

可以从 2 个角度判断：左值**可以取地址、位于等号左边**；而右值**没法取地址，位于等号右边**。

```cpp
int a = 5;
```

-   a可以通过 & 取地址，位于等号左边，所以a是左值。
-   5位于等号右边，5没法通过 & 取地址，所以5是个右值。
  
# 二、什么是左值引用、右值引用

引用本质是别名，可以通过引用修改变量的值，传参时传引用可以避免拷贝，其实现原理和指针类似。

## 左值引用

左值引用大家都很熟悉，**能指向左值，不能指向右值的就是左值引用**：

```cpp
int a = 5;
int &ref_a = a; // 左值引用指向左值，编译通过
int &ref_a = 5; // 左值引用指向了右值，会编译失败
```

**引用是变量的别名，由于右值没有地址，没法被修改，所以左值引用无法指向右值。**

但是，`const` 左值引用是可以指向右值的：

```cpp
const int &ref_a = 5;  // 编译通过
```

## 右值引用

再看下右值引用，右值引用的标志是`&&`，顾名思义，右值引用专门为右值而生，**可以指向右值，不能指向左值**：

```cpp
int &&ref_a_right = 5; // ok
 
int a = 5;
int &&ref_a_left = a; // 编译不过，右值引用不可以指向左值
 
ref_a_right = 6;      // 右值引用的用途：可以修改右值  这里就不好理解了
```

## std::move

把  **右值引用指向左值**

```cpp
int a = 5; // a是个左值
int &ref_a_left = a; // 左值引用指向左值
int &&ref_a_right = std::move(a); // 通过 std::move 将左值转化为右值，可以被右值引用指向
 
cout << a; // 打印结果：5
```

`std::move ` 并没有把一个变量里的内容移动到另一个变量，**唯一的功能是把左值强制转化为右值，单纯的std::move(xxx)不会有性能提升**。

##  左值引用、右值引用本身是左值

**被声明出来的左、右值引用都是左值**。 因为被声明出的左右值引用是有地址的，也位于等号左边。

```cpp
// 形参是个右值引用
void change(int&& right_value) {
    right_value = 8;
}
 
int main() {

    int a = 5; // a是个左值
    int &ref_a_left = a; // ref_a_left 是个左值引用
    int &&ref_a_right = std::move(a); // ref_a_right 是个右值引用
 
	change(a);                    // 编译不过，a是左值，change参数要求右值
    change(ref_a_left);           // 编译不过，左值引用ref_a_left本身也是个左值
    change(ref_a_right);          // 编译不过，右值引用ref_a_right本身也是个左值
     
    change(std::move(a));            // 编译通过
    change(std::move(ref_a_right));  // 编译通过
    change(std::move(ref_a_left));   // 编译通过
 
    change(5); // 当然可以直接接右值，编译通过
     
    cout << &a << ' ';
    cout << &ref_a_left << ' ';
    cout << &ref_a_right;
    
    // 打印这三个左值的地址，都是一样的
}
```

 从表达式`int &&ref = std::move(a)` 可以看出来，move返回的`int &&`是个右值。**右值引用既可以是左值也可以是右值， 作为函数返回值的 && 是右值，直接声明出来的 && 是左值**。 

最后，从上述分析中我们得到如下结论：

1、从性能上讲，左右值引用没有区别，传参使用左右值引用都可以避免拷贝。

2、右值引用可以直接指向右值，也可以通过`std::move`指向左值；而左值引用只能指向左值(const左值引用也能指向左值)。

3、作为函数形参时，右值引用更灵活。虽然 const 左值引用也可以做到左右值都接受，但它无法修改，有一定局限性。


# 三、右值引用和 std::move 的应用场景

##  实现移动语义

在实际场景中，右值引用和 `std::move` 被广泛用于在STL和自定义类中 **实现移动语义，避免拷贝，从而提升程序性能**。 在没有右值引用之前，一个简单的数组类通常实现如下，有`构造函数`、`拷贝构造函数`、`赋值运算符重载`、`析构函数`等。深拷贝/浅拷贝在此不做讲解。

```cpp
class Array {
public:
    Array(int size) : size_(size) {
        data = new int[size_];
    }
     
    // 深拷贝构造
    Array(const Array& temp_array) {
        size_ = temp_array.size_;
        data_ = new int[size_];
        for (int i = 0; i < size_; i ++) {
            data_[i] = temp_array.data_[i];
        }
    }
     
    // 深拷贝赋值
    Array& operator=(const Array& temp_array) {
        delete[] data_;
 
        size_ = temp_array.size_;
        data_ = new int[size_];
        for (int i = 0; i < size_; i ++) {
            data_[i] = temp_array.data_[i];
        }
    }
 
    ~Array() {
        delete[] data_;
    }
 
public:
    int *data_;
    int size_;
};
```

该类的拷贝构造函数、赋值运算符重载函数已经通过使用左值引用传参来避免一次多余拷贝了，但是内部实现要深拷贝无法避免。 这时，有人提出一个想法：是不是可以提供一个`移动构造函数`，把被拷贝者的数据移动过来，被拷贝者后边就不要了，这样就可以避免深拷贝了，

在STL的很多容器中，都实现了以**右值引用为参数**的`移动构造函数`和`移动赋值重载函数`，或者其他函数，最常见的如 `std::vector` 的`push_back`和`emplace_back`。参数为左值引用意味着拷贝，为右值引用意味着移动。

```cpp
class Array {
public:
    ......
 
    // 优雅 移动拷贝
    Array(Array&& temp_array) {
        data_ = temp_array.data_;
        size_ = temp_array.size_;
        // 为防止temp_array析构时delete data，提前置空其data_      
        temp_array.data_ = nullptr;
    }
     
 
public:
    int *data_;
    int size_;
};
```

如何使用：

```cpp
// 例1：Array用法
int main(){
    Array a;
 
    // 做一些操作
    .....
     
    // 左值a，用std::move转化为右值
    Array b(std::move(a));
}
```

##  实例：vector::push_back 使用std::move提高性能

```cpp
// 例2：std::vector和std::string的实际例子
int main() {
    std::string str1 = "aacasxs";
    std::vector<std::string> vec;
     
    vec.push_back(str1); // 传统方法，copy
    vec.push_back(std::move(str1)); // 调用移动语义的 push_back 方法，避免拷贝，str1 会失去原有值，变成空字符串
    vec.emplace_back(std::move(str1)); // emplace_back 效果相同，str1 会失去原有值
    vec.emplace_back("axcsddcas"); // 当然可以直接接右值
}
 
// std::vector方法定义
void push_back (const value_type& val);
void push_back (value_type&& val);
 
void emplace_back (Args&&... args);
```

在 vector 和 string 这个场景，加个`std::move`会调用到移动语义函数，避免了深拷贝。

除非设计不允许移动，STL类大都支持移动语义函数，即`可移动的`。 另外，编译器会**默认**在用户自定义的`class`和`struct`中生成移动语义函数，但前提是用户没有主动定义该类的`拷贝构造`等函数。 **因此，可移动对象在<需要拷贝且被拷贝者之后不再被需要>的场景，建议使用**`std::move`**触发移动语义，提升性能。**

还有些STL类是`move-only`的，比如`unique_ptr`，这种类只有移动构造函数，因此只能移动(转移内部对象所有权，或者叫浅拷贝)，不能拷贝(深拷贝):

```cpp
std::unique_ptr<A> ptr_a = std::make_unique<A>();

std::unique_ptr<A> ptr_b = std::move(ptr_a); // unique_ptr只有‘移动赋值重载函数‘，参数是 && ，只能接右值，因此必须用std::move转换类型

std::unique_ptr<A> ptr_b = ptr_a; // 编译不通过
```
# 四、 完美转发 std::forward

和`std::move`一样，它的兄弟`std::forward`也充满了迷惑性，虽然名字含义是转发，但他并不会做转发，同样也是做类型转换.

与 `std::move` 相比，`std::forward`更强大，`std::move` 只能转出来右值，`std::forward`都可以。

> `std::forward<T>(u)`有两个参数：T 与 u。 a) 当T为左值引用类型时，u将被转换为T类型的左值；  b) 否则u将被转换为T类型右值。  

```cpp

void change2(int&& ref_r) {
    ref_r = 1;
}
 
void change3(int& ref_l) {
    ref_l = 1;
}
 
// change 的入参是右值引用
// 有名字的右值引用是 左值，因此ref_r是左值
void change(int&& ref_r) {
    change2(ref_r);  // 错误，change2 的入参是右值引用，需要接右值，ref_r是左值，编译失败
     
    change2(std::move(ref_r)); // ok，std::move把左值转为右值，编译通过
    change2(std::forward<int &&>(ref_r));  // ok，std::forward的T是右值引用类型(int &&)，符合条件b，因此u(ref_r)会被转换为右值，编译通过
     
    change3(ref_r); // ok，change3的入参是左值引用，需要接左值，ref_r是左值，编译通过
    change3(std::forward<int &>(ref_r)); // ok，std::forward的T是左值引用类型(int &)，符合条件a，因此u(ref_r)会被转换为左值，编译通过
    // 可见，forward可以把值转换为左值或者右值
}
 
int main() {
    int a = 5;
    change(std::move(a));
}
```

