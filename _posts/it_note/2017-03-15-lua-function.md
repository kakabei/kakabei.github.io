---
layout: post
title: lua 函数
date: 2017-03-15 11:16:53
categories: 编程语言
tags: lua  
excerpt: lua function
---

 lua的函数有必要记录可能也就是 可变参数, 后面的命名参数和多个返回值和python基本一样。 

### 可变参数

 Lua 函数可以接受可变数目的参数，和 C 语言类似在函数参数列表中使用三点（...）
表示函数有可变的参数。Lua 将函数的参数放在一个叫 arg 的表中，除了参数以外，arg
表中还有一个域 n 表示参数的个数。

```lua
function g (a, b, ...) end

g(3) a=3, b=nil, arg={n=0}
g(3, 4) a=3, b=4, arg={n=0}
g(3, 4, 5, 8) a=3, b=4, arg={5, 8; n=2}

```
Lua 的函数还一些特性和C不一样的是：它和其他值（数值、字符串）一样，函数可以被存放在变
量中，也可以存放在表中，可以作为函数的参数，还可以作为函数的返回值。

```lua
print = math.sin -- `print' now refers to the sine function
a.p(print(1)) --> 0.841470
```

如函数被嵌套的函数里，它可以访问他外部函数中的变量。这一特性强大编程能力。

另一个特征是：匿名函数。

用表达式创建函数：

```lua
foo = function (x) return 2*x end
```

### 闭包

当一个函数内部嵌套另一个函数定义时，内部的函数体可以访问外部的函数的局部变量，这种特征我们称作词法定界。虽然这看起来很清楚，事实并非如此，词法定界加上第一类函数在编程语言里是一个功能强大的概念，很少语言提供这种支持。

```lua
function newCounter()
	local i = 0
	return function() -- anonymous function
		i = i + 1
		return i
	end
end

c1 = newCounter()
print(c1()) --> 1
print(c1()) --> 2

```
i不是全局变量也不是局部变量，我们称作外部的局部变量（external local variable）或者 upvalue。

匿名函数使用 upvalue i 保存他的计数，当我们调用匿名函数的时候 i 已经超出了作用范围，因为创建 i 的函数 newCounter 已经返回了。然而 Lua 用闭包的思想正确处理了这种情况。简单的说闭包是一个函数加上它可以正确访问的 upvalues。如果我们再次调
用 newCounter，将创建一个新的局部变量 i，因此我们得到了一个作用在新的变量 i 上的
新闭包。

```lua
c2 = newCounter()
print(c2()) --> 1
print(c1()) --> 3
print(c2()) --> 2

```
技术上来讲，闭包指值而不是指函数，函数仅仅是闭包的一个原型声明；尽管如此，
在不会导致混淆的情况下我们继续使用术语函数代指闭包。

闭包在上下文环境中提供很有用的功能，如高级函数（sort）的参数；作为函数嵌套的函数（newCounter）。这一机制使得我们可以在 Lua 的函数世界里组合出奇幻的编程技术。闭包也可用在回调函数中。

### 全局函数和局部函数

局部函数就是在函数前加一个 local。

局部函数的两种方式：

1. 方式一

```lua
local f = function (...)
...
end
local g = function (...)
...
f() -- external local `f' is visible here
...
end
```

2. 方式二

```lua
local function f (...)
...
end
```

有一点需要注意的是在声明递归局部函数的方式。
要提前定义local。

```lua

local fact
fact = function (n)
	if n == 0 then
		return 1
	else
		return n*fact(n-1)
	end
end
```

### 尾调用（Proper Tail Calls）

尾调用是一种类似在函数结尾的 goto 调用，当函数最后一个动作是调用另外一个函
数时，我们称这种调用尾调用。例如：

```lua
function f(x)
	return g(x)
end
```

g 的调用是尾调用。

例子中 f 调用 g 后不会再做任何事情，这种情况下当被调用函数 g 结束时程序不需
要返回到调用者 f；所以尾调用之后程序不需要在栈中保留关于调用者的任何信息。

如下三种不是尾调函数.

```lua

r eturn g(x) + 1 -- must do the addition
return x or g(x) -- must adjust to 1 result
return (g(x)) -- must adjust to 1 result
```