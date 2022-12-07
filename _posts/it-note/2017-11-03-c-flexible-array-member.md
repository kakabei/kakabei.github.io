---
layout: post
title: c 柔性数组
date: 2017-11-03 16:12:15
categories: c++  
tags: c++ 技术笔记
excerpt: 柔性数组,这种代码结构产生于对动态结构体的需求,在工作中可以被灵活的运用
---

# 柔性数组

**柔性数组成员**（flexible array member）也叫**伸缩性数组成员**，这种代码结构产生于对动态结构体的需求。在日常的编程中，有时候需要在结构体中存放一个长度动态的字符串，鉴于这种代码结构所产生的重要作用，C99 甚至把它收入了标准中：

> As a special case, the last element of a structure with more than one named member may have an incomplete array type; this is called a flexible array member.
    
-   柔性数组成员必须定义在结构体里面且为最后元素；
-   结构体中不能单独只有柔性数组成员；
-   柔性数组不占内存。

```c
struct test { 
    short len;   // 必须至少有一个其它成员     
    char arr[]; // 柔性数组必须是结构体最后一个成员.
};
```

柔性数组是 C99 标准引入的特性，所以当你的编译器提示不支持的语法时，请检查你是否开启了 C99 选项或更高的版本支持。

在一个结构体的最后，申明一个长度为空的数组，就可以使得这个结构体是可变长的。

对于编译器来说，此时长度为 0 的数组并不占用空间，因为数组名本身不占空间，它只是一个偏移量，数组名这个符号本身代表了一个不可修改的地址常量。

对于柔性数组的这个特点，很容易构造出变成结构体，如缓冲区，数据包等等。

其实柔性数组成员在实现跳跃表时有它特别的用法，在 Redis的 SDS 数据结构中和跳跃表的实现上，也使用柔性数组成员。它的主要用途是为了满足需要变长度的结构体，为了解决使用数组时内存的冗余和数组的越界问题。

# 例子

**结构体**：

```c
// 柔性数组 
struct soft_buffer {
    int    len;
    char   data[0]; 
};
```


数据结构大小 = `sizeof(struct soft_buffer)` = `sizeof(int)`，这样的变长数组常用于网络通信中构造不定长数据包, 不会浪费空间浪费网络流量。

**申请内存**：
```c++

struct soft_buffer * softbuffer = (struct soft_buffer *)malloc(sizeof(struct soft_buffer) + sizeof(char) * CUR_LENGTH);

if (softbuffer != NULL) {
	softbuffer->len = CUR_LENGTH;
	memcpy(softbuffer->data, "softbuffer test", CUR_LENGTH); 
	printf("%d, %s\n", softbuffer->len, softbuffer->data);
}
```

**释放内存**：

```c++
free(softbuffer); 
softbuffer = NULL;
```

# 优点

使用柔性数组的优点：

1、减少内存碎片，由于结构体的柔性数组和结构体成员的地址是连续的，即可一同申请内存，因此更大程度地避免了内存碎片。

2、成员本身不占结构体空间，因此，整体而言，比普通的数组成员占用空间要会稍微小点。

缺点：对结构体格式有要求，必要放在最后，不是唯一成员。







----
参考资料：

1、[C语言解柔性数组是什么？](https://cloud.tencent.com/developer/article/1764391)