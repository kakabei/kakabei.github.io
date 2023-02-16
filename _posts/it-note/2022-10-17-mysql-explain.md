---
layout: post
title: mysql explain 命令详解
date: 2022-10-17 16:12:15
categories: db  
tags: mysql 
excerpt: 在工作中 explain 命令用于分析 sql 语句的性能效率。如果 sql 比较复杂，最好分析一下语句，方便优化 
---


	
explain 命令主要是为了分析 SQL 执行的效率。 explain 显示了 mysql 如何使用索引来处理 select 语句以及连接表，可以帮助选择更好的索引和写出更优化的查询语句。

使用方法： 在 SQL 语句前加上 explain  命令。 

```sql
 explain select * from user where name = 'root';
```

![](/assets/db/mysql-2022-10-18_16-46-46.png)

说明：
-  id：执行编号，标识 select 所属的行。
-   select_type：select 查询的类型。
-   table：查询的是哪个表。
-   partitions：匹配的分区。
-   **type：关联类型，或者访问类型。**
-   possible_keys：该查询可以选用的索引。
-   key：该查询选用的索引。
-   key_len：索引中使用的字节数。
-   ref：显示上述表的连接匹配条件，即哪些列或常量被用于查询索引列上的值。
-   rows：估计为了找到所需行而要读取的行数。
-   filtered：按表条件过滤的行的百分比。
-   **Extra：额外的信息。**

## type

对表访问方式，表示MySQL在表中找到所需行的方式，又称“访问类型”。

常用的类型有： **ALL < index < range < ref < eq_ref < const < system < NULL（从左到右，性能从差到好）**

- ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行。
- index: Full Index Scan，index与ALL区别为index类型只遍历索引树。
- range:只检索给定范围的行，使用一个索引来选择行。
- ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值。
- eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件。
- const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量，system是const类型的特例，当查询的表只有一行的情况下，使用system。
- NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。


##  Extra

这一列展示的是额外信息。常见的重要值如下：

- distinct : 一旦 mysql 找到了与行相联合匹配的行，就不再搜索了。
- Using index ：这发生在对表的请求列都是同一索引的部分的时候，返回的列数据只使用了索引中的信息，而没有再去访问表中的行记录。是性能高的表现。
- Using where ：mysql 服务器将在存储引擎检索行后再进行过滤。就是先读取整行数据，再按 where 条件进行检查，符合就留下，不符合就丢弃。
- Using temporary：mysql 需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化。
- Using filesort ：mysql 会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。此时mysql会根据联接类型浏览所有符合条件的记录，并保存排序关键字和行指针，然后排序关键字并按顺序检索行信息。这种情况下一般也是要考虑使用索引来优化的。

---
参考资料：

[1、MySQL _Explain_命令详解：type列详解及案例分析](https://zhuanlan.zhihu.com/p/358920539)

[2、MySQL之EXPLAIN命令详解](https://zhuanlan.zhihu.com/p/381852677)

[3、EXPLAIN Join Types](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-join-types)
