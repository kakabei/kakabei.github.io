---
layout: post
title: c++ 类型转换时溢出
date: 2019-10-25 21:00:30
categories: c++ 
tags: c++ bug 工作经验
excerpt: 工作中出现的 bug, 类型转换时溢出,很隐蔽
---
几天出现过一个 bug，很隐蔽, 记录一下。

```c++
uint64_t lastTime; // 毫秒
uin32_t nowTime = time(); 
if (lastTime <  (uint64_t)(nowTime*1000)){ // bug

}

if (lastTime <  (uint64_t)nowTime*1000){ // ok

}

```
这个逻辑判断是有问题的。nowTime是32位 *1000后可能就溢出了，然后再转成64位，结果已经被截断了。