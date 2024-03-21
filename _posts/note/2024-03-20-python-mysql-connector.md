---
layout: post
title: python mysql 插入更新一些特殊的字符
date: 2024-03-09 9:08:12
categories: 工作日志
tags: python 
excerpt: python mysql 使用参数化查询处理特殊字符
---

写了一个脚本，把一个mysql 表的中数据从一个表更新到另一个表中。其他有字段是路径，包含有 `/` 和  `'`  等字符。

第一次的做法是：

```python 
    sql = "update t_softname set icon=%s, start_cmd=%s where name=%s".format(icon, link, name)
    cursor.execute(sql) 
```

却发现了问题，就是在遇到特殊的字符会被转义。如：写入的路径后会没了斜杠 `\`。或遇到 `'` 出现 sql 解析异常。 

这是字符串被转义导致的。 

要**使用参数化查询**才可以，如下：

```python 

    sql = "update t_softname set icon=%s, start_cmd=%s where name=%s"
    cursor.execute(sql,(icon, link, name))  
```