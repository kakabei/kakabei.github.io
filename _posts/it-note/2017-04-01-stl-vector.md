---
layout: post
title: stl vector 笔记
date: 2017-04-01 21:12:15
categories: c++
tags: stl  
excerpt: stl vector 笔记备忘
---

vector 是线性容器。

它的元素是严格按照线性序列排序，存储在一连续的存储空间中。这也是意味着不仅可以用使用迭代器，还可以使用指针的偏移方式访问。
vector的优点如下：
 - 可以使用下标访问个别的元素。
 - 迭代器可以按照不同的方式遍历容器
 - 可以在窗口在末尾增加或删除元素。ss

1、虽然窗口在在自动处理容量的大小时会消耗更多的内存，但是容器能和数组一样的性能，而且能很好的地调整存储空间大小。

2、和其他标准的顺序容器相比， vector能更有效访问窗口在内的元素和末尾添加和删除元素，而在其他的位置添加和删除元素，vector则不及其他顺序容器。

3、在迭代器和引用的也不比list支持的好。

4、容器的大小和窗口在的容量是有区别的，大小是元素的，容量是分配的内存大小。大小vector::size() ，容量是vector::capacity()

5、可用 `for_echo()` 遍历。如： 

```c++
for_echo(ivetor.begin(), ivector.end(), print) // print是回调函数   void print(int n );

```

6、`<algorithm>`中的sort函数，对象是vector。    

7、vector 查找，`std::find(vector.begin(), vector.end(), 3);`

8、vector 删除，有两种方式 erase()指定位置  或pop_back()最后一个。

9、当进行添加是，可以用 `insert()` 或 `push_bask()`，当容量不够时，会触发动态扩容，此时，它的性能会下降，所以，一般要先预估vector的大小，先 `vector::reserver()`。避免出现动态扩容。

10、使用“交换技巧”来修整 vector 过剩的空间/内存。`vector<int>(ivec).swap(ivec)`。`vector<int>(ivec)` 建立一个临时 的vector，它是ivec的一个拷贝。vector的拷贝构造函数只分配拷贝的元素所需要的大小。

11、用swap方法强行释放vector所点内存.。这个地方应该怎么解释？

```c
tmplate< class T> void clearVector( vector <T> &v )
{
     vector<T> vtTemp;
     vtTemp,swap(v);
}

vector<int> v ; 
v.push_back(1);

```
或  `vector<int>().swap(v)`

或 `v.swap(vector<int>());`

或 `{   std::vector<int> tmp = v; v.swap(tmp); }　　//大括号让tmp退出｛｝时自动析构。`


12、 关于vector删除的问题。有一些地方要注意。遍历一个vector找到对应的元素删除的正常做法。 对于容器的删除都要小心。

```c++

for (auto iter = vector.begin(); iter != vector.end(); iter++)
{
     if (*iter == 3 )   vector.ease(iter);
｝

```

1）这个代码有一个严重的错误。当ease(iter)后，iter就成了一个野指针。再对它iter++,就会有问题。

2）正常 的做法。

```c++
     vector<int>::iterator iter=vec.begin();
     for (; iter!= vec.end();)
     {
          iter = vec.erase(iter); 
     }
     else
     { 
       ++iter;
     }
```     

13、 c++11 新增的方法：

```c++

vector::cbegin();  // 返回的是const iterator 
vector::cend();
vector::crbegin();
vector::crend():

vector::data();  // 返回第一个元素的指针
vector::emplace();  // emplace函数的参数根据元素类型而变化，参数必须与元素类型的构造函数相匹配.     
vector::emplace_back(); 
vector::shrink_to_fit();  // 减少vector的容量到合适的大小。

```
  

