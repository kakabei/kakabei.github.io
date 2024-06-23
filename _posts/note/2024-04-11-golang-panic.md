---
layout: post
title: golang painc 发生了
date: 2024-04-11 9:08:12
categories: 开发语言
tags: golang  
excerpt: 无论有意还是无意，painc 就是发生了
---

```go
	Config1 := mapConf["game"].(map[string]interface{})
	for key, value := range confMap {
		Config1[key] = value
	}
```
mapConf["game"]为nil：如果mapConf["game"]是 nil，那么在尝试进行类型断言时也会导致 panic。

改成：

```go
    Config1 := make(map[string]interface{})
	for key, value := range confMap {
		Config1[key] = value
	}
	mapConf["game"] = Config1
```