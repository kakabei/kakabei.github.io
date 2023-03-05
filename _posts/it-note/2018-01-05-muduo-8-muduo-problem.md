---
layout: post
title: muduo笔记  muduo 在网上看到一些疑问
date: 2018-01-05 21:22:12
categories: muduo
tags:  系统编程 muduo 技术学习笔记 
excerpt: muduo 在网上看到一些疑问
---

大致看完muduo关于网络这一部分了， 其他的都是一些基本库。在base这个目录下。

也在网上看看别对muduo的一些评价和疑问。也可学习到很多，毕竟有一些问题自己没注意到。

[知乎上muduo的一些问题](https://www.zhihu.com/topic/20013250/hot)

[muduo中，本来可以直接调用的函数，为什么要扔到 EventLoop 中呢？](https://www.zhihu.com/question/59576980/answer/166730296)

[muduo库的很多返回值为空的函数的最后一行都有类似 (void) n的一句， 为什么要这样做？](https://www.zhihu.com/question/24311085)

你在linux多线程编程中1.12说，member data 的mutex 不能保护析构，TcpClient 的析构函数使用 MutexLock 没有问题吗？

[多线程编程中什么情况下需要加 volatile？](https://www.zhihu.com/question/31459750/answer/52061391)

[类初始化时 this 指针何时生成，在构造函数中如何确保 this 指针有效？](https://www.zhihu.com/question/279734963/answer/409009408)

[对象池的一个 race condition]https://zhuanlan.zhihu.com/p/30522095

这就是一个，看代码时就是没能注意到。

---
[muduo库在实际项目中使用的人多吗？](https://www.zhihu.com/question/24590359)

这个是关于一些muduo的讨论，很有意思。可能用于生产环境的人真的不多吧。但muduo的思想对于学习网络库还是很有借鉴的作用。像360的[evpp](https://github.com/Qihoo360/evpp)就是参考了muduo。

长时间在源码代中，特别还看陈硕的书。很容易眼障， 看不出不好的地方。像这里提到了水木网友的文章。不知道原文章的出处[https://pastebin.com/8S7CjdSb](https://pastebin.com/8S7CjdSb)。同时要看一下下面回复的一些讨论。每个人的观点不可能一样的。各有所爱吧。


[当应用程序调用Send之后怎么判断对方是否成功接收?](https://www.zhihu.com/question/25016042/answer/29798924?group_id=783954989#comment-59736874)
这个地方关于在tcp中, send/recv能否正确写好的问题。 对muduo的一些看法，也是超出了我的见识啊。又学习了。对于TCP的理解还是深到那个地步啊。看那个水手应该是个牛人。





