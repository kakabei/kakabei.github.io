---
layout: post
title: lua metatables 元表
date: 2017-03-15 21:12:15
categories: 编程语言 lua
tags: lua  
excerpt: lua metatables 允许改变 table 的行为。
---

> Metatables 允许我们改变 table 的行为，例如，使用 Metatables 我们可以定义 Lua 如
何计算两个 table 的相加操作 a+b。

这种方式很类似于C++中对运算符的重载。

Lua 中的每一个表都可以有它自己的 Metatable。一般情况下 Lua默认创建一个不带 metatable 的新表。

用getmetatable(table) 可以获取这个表的Metatable。

用setmetatable(table, metatable) 对一个表设置Metatable。

metatable 算术运算符域名 有__add(加)、__mul(乘)、__sub(减)、__div(除)、__unm(负)、__pow(幂)，我们也可以定义__concat 定义连接行为。

metatable 关系运算符 ：__eq（等于），__lt（小于） ，和__le（小于等于）。

Lua 选择 metamethod 的原则：如果第一个参数存在带有__add 域的 metatable，Lua
使用它作为 metamethod，和第二个参数无关；
否则第二个参数存在带有__add 域的 metatable， Lua 使用它作为 metamethod 否则报
错。

如果想保护你的集合使其使用者既看不到也不能修改 metatables。可以
对 metatable 设置了__metatable 的值， getmetatable 将返回这个域的值，而调用 setmetatable
将会出错：

```lua
Set.mt.__metatable = "not your business"
s1 = Set.new{}
print(getmetatable(s1)) --> not your business
setmetatable(s1, {})
stdin:1: cannot change protected metatable
```

table访问的元方法： 字段: __index __newindex

__index:  查询：访问表中不存的字段  rawget(t, i)

__newindex： 更新：向表中不存在索引赋值  rawswt(t, k, v)

有默认值的表:

在一个普通的表中任何域的默认值都是 nil。很容易通过 metatables 来改变默认值：

```lua
function setDefault (t, d)
	local mt = {__index = function () return d end}
	setmetatable(t, mt)
end

tab = {x=10, y=20}
print(tab.x, tab.z) --> 10 nil

setDefault(tab, 0)
print(tab.x, tab.z) --> 10 0
```

监控表:

```lua

-- create private index
local index = {}

-- create metatable
local mt = {
	__index = function (t,k)
		print("*access to element " .. tostring(k))
		return t[index][k] -- access the original table
	end

	__newindex = function (t,k,v)
		print("*update of element " .. tostring(k) .. " to ".. tostring(v))
		t[index][k] = v -- update original table
	end
}

function track (t)

	local proxy = {}
	proxy[index] = t

	setmetatable(proxy, mt)
	return proxy
end
```

只读表:

```lua

function readOnly (t)
	local proxy = {}
	local mt = { -- create metatable
	__index = t,
	__newindex = function (t,k,v)
		error("attempt to update a read-only table", 2)
	end
	}
	setmetatable(proxy, mt)
	return proxy
end

days = readOnly{"Sunday", "Monday", "Tuesday", "Wednesday",
"Thursday", "Friday", "Saturday"}


print(days[1]) --> Sunday
days[2] = "Noday"
-- stdin:1: attempt to update a read-only table

```