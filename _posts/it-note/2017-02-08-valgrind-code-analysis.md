---
layout: post
title: 用valgrind做一次性能分析
date: 2017-02-08 21:12:15
categories: 性能分析
tags: valgrind  
excerpt: 用valgrind可以分析出代码的热点
---

有一次用valgrind对代码的热点进行分析。发现有一个函数的被调用百分比比较高。

关键的函数是CGameBuffMgr::process()

```c++
bool CGameBuffMgr::process (uint64 uTick,uint64 uTime,uint32 uSecond)
{
	if(m_uProcessTick > uTick)
		return true;

	if(!m_uProcessTick)
		m_uProcessTick = uTick + 100;

	POOL_BUFF::iterator _pos;
	m_poolBuff.getHead(_pos);
	while (!m_poolBuff.isTail(_pos))
	{
		CGameBuff*	pBuff = m_poolBuff.getNext(_pos);
		if (!pBuff)
			continue;
		pBuff->process(uTick,uTime,uSecond);
		if (pBuff->isDelete())
			delBuff(pBuff,true);
	}

	return true;
}
```
process是由定时器调起，所以比较多是正常的。

用 valgrind 做代码性能分析。

```sh
/usr/bin/valgrind --tool=callgrind  --trace-children=yes  /data/game_server/game_server
```

得到：callgrind.out.6578 文件。再用 kcachegrind进行分析。如下：

![](/assets/code-analysis/valgrind-stl-1.png) 

CGameBuffMgr::process中的 getHead消耗比较大。感觉可能有问题。

而getHead是这样的：

```c++
template <typename T, typename _ValType, int COUNT>
inline void	CMemoryPool<T,_ValType,COUNT>::getHead(iterator& pos)
{
	try
	{
		//CCritLocker lock(m_csLock);
		pos = m_UseList.begin();
	}
	catch (...)
	{
	}
}

```
getHead中只是对 hash_map进行begin操作。
从图上看hash_map的begin消耗也很大。问题可能出在这个地方。

![](/assets/code-analysis/valgrind-stl-12.png) 


在 [http://blog.csdn.net/tototony/article/details/5689882](http://blog.csdn.net/tototony/article/details/5689882)这篇文章中，可以了解一些hash_map的原理。
begin() 为了获得第一个元素，就在hashtable表中遍历，hashtale就是一个vector。
所以说 一调用begin(), 说是遍历hashtable

```c++
iterator hashtable::begin()
{
	for(size_type n = 0; n < _M_buckets.size(); ++__n)
		if(_M_buckets[__n])
			return iterator(_M_buckets[__n], this);
	
	return end();
}
```

找到了就返回，找到就会遍历完整个hashtable表，再返回end()。
我想这就是为什么hash_map中的 begin() 效率不行的原因吧。

最后，就动手把hash_map改成了map　stl中的map其实就是红黑树。
改动后 重新跑一下valgrind。 得到如下图。已经没有getHead了。

![](/assets/code-analysis/valgrind-stl-2.png) 

所以，如果要用到遍历的应尽量避免用hash_map。