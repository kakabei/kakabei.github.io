---
layout: post
title: mysql 常用语句
date: 2015-05-22 11:08:12
categories: db
tags:  mysql 
excerpt: mysql 数据库常用 sql 语句，工作中用到频率比较高 
---

# 一、数据库

##  1.1  登录

```shell
mysql -h 127.0.0.1 -u <用户名> -p<密码>.   ##  默认用户名<root>，-p 是密码，参数后面不需要空格

mysql -D 所选择的数据库名 -h 主机名 -u 用户名 -p
```

## 1.2  退出
```shell 
mysql> exit  ## 退出 
```

## 1.3 数据相关信息

``` shell 
mysql> status;   ## 显示当前mysql的version的各种信息

mysql> select version();  ## 显示当前mysql的version信息

mysql> show global variables like 'port';  ## 查看MySQL端口号
```

## 1.4 数据库操作

```mysql 
--  创建一个数据库
create database database_name character set gbk; 

-- 删除数据库
drop database database_name;                     

-- 显示数据库列表
show databases;                                  

-- 切换数据库
use database_name;                               	
```


# 二、表

## 2.1 表操作

```mysql 
-- 显示 samp_db 下面所有的表名字
show tables;                 

-- 显示数据表的结构
describe table_name;         

-- 清空表中记录
delete from table_name;       

-- 用于删除数据库中的现有表。
drop table table_name;        

-- 用于删除表内的数据，但不删除表本身。
truncate table table_name;   

-- 修改表名
alter table table_name rename new_table_name;
```

## 2.2 表创建

```mysql 
CREATE TABLE `UserInfo` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '编号ID',
  `name` varchar(64) NOT NULL DEFAULT '' COMMENT '用户名',
  `create_time` datetime NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '更新时间',
  `extendInfo` varchar(1024) NOT NULL DEFAULT '' COMMENT '扩展字段',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

表的一些属性说明：

- `NULL`：数据列可包含 NULL 值；
- `NOT NULL`：数据列不允许包含 NULL 值；
- `DEFAULT`：默认值；
- `PRIMARY KEY`：主键；
- `AUTO_INCREMENT`：自动递增，适用于整数类型；
- `UNSIGNED`：是指数值类型只能为正数；
- `CHARACTER SET name`：指定一个字符集；
- `COMMENT`：对表或者字段说明；

## 2.3 列的操作

```mysql 
-- 添加列
alter table 表名 add 列名 列数据类型 [after 插入位置];

-- 在后添加 birthday 
alter table Userinfo add birthday varchar(100) not null default "" date after age;

-- 修改列
alter table 表名 change 列名称 列新名称 新数据类型;

-- 将 name 列的数据类型改为 char(16): 
alter table Userinf change name name char(16) not null;

-- 删除列
alter table 表名 drop 列名称;

-- 删除 birthday 这一列
alter table Userinfo drop birthday;
```


# 三、常用语句

## 3.1 SELECT

```mysql 
select id, name from UserInfo where id = 111; 
```

## 3.2 UPDATE


```mysql 
-- update语句设置字段值为另一个结果取出来的字段
update UserInfo set name='南山肯德鸡'  where id = 111; 
```

## 3.3 INSERT

```mysql 
insert into UserInfo(name, create_time，update_time) value ('夏有凉风'， '2020-11-10 08:33:21', '2020-11-10 08:33:21');

-- 从一个表的数据插入到另外一个表
insert into live (user_id, name) SELECT m.user_id, m.name FROM moot m where m.id=111;


-- 如不存在则插入，如存在则更新
INSERT INTO `charger` (`id`,`type`,`create_at`,`update_at`) VALUES (3,2,'2017-05-18 11:06:17','2017-05-18 11:06:17') ON DUPLICATE KEY UPDATE `id`=VALUES(`id`), `type`=VALUES(`type`), `update_at`=VALUES(`update_at`);
```


## 3.4  DELETE

现实开发实践中，业务上一般不允许删除。DBA 要把这个权限收回。

```mysql
-- 清空表
delete from  UserInfo
-- 或
delete * from  UserInfo

-- 删除指定的一行
delete from  UserInfo  WHERE id = 111; 
```


## 3.5  ORDER BY

order by   用于按升序或降序对结果集进行排序。 默认按 asc 。  asc 升序排序, desc 降序排序。 

```mysql 

-- 升序
select * from UserInfo order by  id asc; 

-- 降序
select * from UserInfo order by id desc;
```


## 3.6 GROUP BY

将具有相同值的行分组到汇总。

```mysql 
-- 分组汇总不同性别的人数
select male, count(id) from UserInfo group by male; 
```


## 3.7  UNION

合并两个或多个 SELECT 语句的结果集。

```mysql 
select name from Employees_ZH union select name from Employees_USA; 
```


## 3.8  JOIN

JOIN 子句用于根据两个或多个表之间的相关列组合来自两个或多个表的行。

- `INNER JOIN` : 两个表都存在的行，才返回。
- `LEFT JOIN`:  右表没有，左表有，也要返回。
- `RIGHT JOIN`: 左表没有，右表有，也要返回。
- `FULL JOIN`: 只要其中一个表中有，就返回行。 (MySQL 是不支持的，通过 `LEFT JOIN + UNION + RIGHT JOIN` 的方式 来实现)


## 2.9  FULL OUTER JOIN

当左（表1）或右（表2）表记录中存在匹配时，关键字返回所有记录

# 四、SQL函数

## 4.1 COUNT

统计满足条件的行数。

```mysql 
select male, count(id) from UserInfo group by male; 
```

## 4.2 AVG

数值列的平均值。

```mysql
SELECT AVG(列名称) FROM 表名称 WHERE 条件;
```

## 4.3 SUM

数值列的总和

```mysql 
SELECT SUM(列名称) FROM 表名称 WHERE 条件;
```

## 4.4 MAX

所选列的最大值

```mysql 
SELECT MIN(列名称) FROM 表名称 WHERE 条件;
```

## 4.5 MIN

所选列的最小值

```mysql 
SELECT MIN(列名称) FROM 表名称 WHERE 条件;
```


# 五、索引

## 5.1  普通索引

```mysql 
--  添加索引 普通索引(INDEX)
ALTER TABLE `表名字` ADD INDEX 索引名字 ( `字段名字` )

-- 直接创建索引
CREATE INDEX index_user ON Userinfo(name)

-- 修改表结构的方式添加索引
ALTER TABLE table_name ADD INDEX index_name ON (column(length))

-- 给 表中的 name 字段 添加普通索引
ALTER TABLE `Userinfo` ADD INDEX index_name (name)

-- 创建表的时候同时创建索引
CREATE TABLE `user` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL ,
    `content` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    PRIMARY KEY (`id`),
    INDEX index_name (title(length))
)

-- 删除索引
DROP INDEX index_name ON table
```

## 5.2  主键索引

```mysql 
ALTER TABLE `表名字` ADD PRIMARY KEY ( `字段名字` )
```

## 5.3 唯一索引

```mysql
ALTER TABLE `表名字` ADD UNIQUE (`字段名字`)
```

## 5.4  全文索引

```mysql
ALTER TABLE `表名字` ADD FULLTEXT (`字段名字`)
```

## 5.5  添加多列索引

```mysql 
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3`)
```

# 六、其他

## 6.1 删除重复记录

```mysql 
-- 查找表中多余的重复记录，重复记录是根据单个字段（peopleId）来判断
select * from people where peopleId in (select peopleId from people group by peopleId having count(peopleId) > 1)

-- 删除表中多余的重复记录，重复记录是根据单个字段（peopleId）来判断，只留有rowid最小的记录
delete from people 
where peopleId in (select peopleId from people group by peopleId having count(peopleId) > 1)
and rowid not in (select min(rowid) from people group by peopleId having count(peopleId )>1)

-- 查找表中多余的重复记录（多个字段）
select * from vitae a
where (a.peopleId,a.seq) in (select peopleId,seq from vitae group by peopleId,seq having count(*) > 1)

-- 删除表中多余的重复记录（多个字段），只留有rowid最小的记录
delete from vitae a
where (a.peopleId,a.seq) in (select peopleId,seq from vitae group by peopleId,seq having count(*) > 1) and rowid not in (select min(rowid) from vitae group by peopleId,seq having count(*)>1)

-- 查找表中多余的重复记录（多个字段），不包含rowid最小的记录
select * from vitae a
where (a.peopleId,a.seq) in (select peopleId,seq from vitae group by peopleId,seq having count(*) > 1) and rowid not in (select min(rowid) from vitae group by peopleId,seq having count(*)>1)

```

## 6.2 批量创建分表

```shell
for i in {0..1023}
do

mysql -h127.0.0.1 -uadmin -padmin123 -P3358 hgame  -e "CREATE TABLE UserInfo$i (

  id bigint(20) unsigned NOT NULL COMMENT 'SKU ID',
  createi_time varchar(255) NOT NULL DEFAULT '' COMMENT '导入时间',
  ext varchar(255) NOT NULL DEFAULT '',
  PRIMARY KEY (id)

) ENGINE=InnoDB DEFAULT CHARSET=utf8";

done
```


---

1、[https://github.com/jaywcjlove/mysql-tutorial/blob/master/docs/21-minutes-MySQL-basic-entry.md](https://github.com/jaywcjlove/mysql-tutorial/blob/master/docs/21-minutes-MySQL-basic-entry.md)

2、[https://www.w3school.com.cn/sql/index.asp](https://www.w3school.com.cn/sql/index.asp)