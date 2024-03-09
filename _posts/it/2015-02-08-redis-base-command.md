---
layout: post
title: redis 常用命令
date: 2015-02-08 11:08:12
categories: db
tags:  redis
excerpt: redis 常用命令汇总
---

# 连接控制

- `QUIT` 关闭连接
- `AUTH` (仅限启用时)简单的密码验证

# 适合全体类型的命令

- `EXISTS key` 判断一个键是否存在;存在返回 1;否则返回 0;
- `DEL key` 删除某个 key,或是一系列 key;DEL key1 key2 key3 key4
- `TYPE key` 返回某个 key 元素的数据类型 ( none:不存在,string:字符,list,set,zset,hash)
- `KEYS pattern` 返回匹配的 key 列表 (KEYS foo*:查找 foo 开头的 keys)
- `RANDOMKEY` 随机获得一个已经存在的 key，如果当前数据库为空，则返回空字符串
- `RENAME oldname newname` 更改 key 的名字，新键如果存在将被覆盖
- `RENAMENX oldname newname` 更改 key 的名字，如果名字存在则更改失败
- `DBSIZE` 返回当前数据库的 key 的总数
- `EXPIRE` 设置某个 key 的过期时间(秒),(EXPIRE bruce 1000:设置 bruce 这个 key1000 秒后系统 自动删除)注意:如果在还没有过期的时候，对值进行了改变，那么那个值会被清除。
- `TTL` 查找某个 key 还有多长时间过期,返回时间秒
- `SELECT index` 选择数据库
- `MOVE key dbindex` 将指定键从当前数据库移到目标数据库 dbindex。成功返回 1;否则返回 0(源 数据库不存在 key 或目标数据库已存在同名 key);
- `FLUSHDB` 清空当前数据库中的所有键
- `FLUSHALL` 清空所有数据库中的所有键
  
# 处理字符串的命令

- `SET key value` 给一个键设置字符串值。`SET keyname datalength data` (`SET bruce 10 paitoubing` : 保存 key 为 burce,字符串长度为 10 的一个字符串 paitoubing 到数据库)，data 最大不可超过 1G。 GET key 获取某个 key 的 value 值。如 key 不存在，则返回字符串“nil”;如 key 的值不为字符串 类型，则返回一个错误。
- `GETSET key value` 可以理解成获得的 key 的值然后 SET 这个值，更加方便的操作 (`SET bruce 10 paitoubing`,这个时候需要修改 bruce 变成 1234567890 并获取这个以前的数据 `paitoubing,GETSET bruce 10 1234567890`)
- `MGET key1 key2 ... keyN` 一次性返回多个键的值
- `SETNX key value SETNX` 与 SET 的区别是 SET 可以创建与更新 key 的 value，而 SETNX 是如果 key 不存在，则创建 key 与 value 数据
- `MSET key1 value1 key2 value2 ... keyN valueN` 在一次原子操作下一次性设置多个键和值
- `MSETNX key1 value1 key2 value2 ... keyN valueN` 在一次原子操作下一次性设置多个键和值(目标键不存在情况下，如果有一个以上的 key 已存在，则失败)
- `NCR key` 自增键值
- `INCRBY key integer` 令键值自增指定数值
- `DECR key` 自减键值
- `DECRBY key integer` 令键值自减指定数值

# 处理 lists 的命令

- `RPUSH key value` 从 List 尾部添加一个元素(如序列不存在，则先创建，如已存在同名 Key 而非 序列，则返回错误)
- `LPUSH key value` 从 List 头部添加一个元素
- `LLEN key` 返回一个 List 的长度
- `LRANGE key start end` 从自定的范围内返回序列的元素 (`LRANGE testlist 0 2`; 返回序列 testlist 前 0 1 2 元素)
- `LTRIM key start end` 修剪某个范围之外的数据 (`LTRIM testlist 0 2`; 保留 0 1 2 元素，其余的删除)
- `LINDEX key index` 返回某个位置的序列值(LINDEX testlist 0;返回序列 testlist 位置为 0 的元素) LSET key index value 更新某个位置元素的值
- `LREM key count value` 从 List 的头部(count 正数)或尾部(count 负数)删除一定数量(count) 匹配 value 的元素，返回删除的元素数量。
- `LPOP key` 弹出 List 的第一个元素
- `RPOP key` 弹出 List 的最后一个元素
- `RPOPLPUSH srckey dstkey` 弹出 _srckey_ 中最后一个元素并将其压入 _dstkey_头部，key 不存在 或序列为空则返回“nil”
  

# 处理集合(sets)的命令(有索引无序序列)

- `SADD key member` 增加元素到 SETS 序列,如果元素(membe)不存在则添加成功 1，否则失败 0;(`SADD testlist 3 \n one`)
- `SREM key member` 删除 SETS 序列的某个元素，如果元素不存在则失败 0，否则成功 1(`SREM testlist 3 \N one`)
- `SPOP key` 从集合中随机弹出一个成员
- `SMOVE srckey dstkey member` 把一个 SETS 序列的某个元素 移动到 另外一个 SETS 序列 (`SMOVE testlist test 3\n two`;从序列 testlist 移动元素 two 到 test 中，testlist 中将不存在 two 元素) SCARD key 统计某个 SETS 的序列的元素数量
- `SISMEMBER key member` 获知指定成员是否存在于集合中
- `SINTER key1 key2 ... keyN` 返回 key1, key2, ..., keyN 中的交集
- `SINTERSTORE dstkey key1 key2 ... keyN` 将 key1, key2, ..., keyN 中的交集存入 dstkey 
- `SUNION key1 key2 ... keyN` 返回 key1, key2, ..., keyN 的并集
- `SUNIONSTORE dstkey key1 key2 ... keyN` 将 key1, key2, ..., keyN 的并集存入 dstkey
- `SDIFF key1 key2 ... keyN` 依据 key2, ..., keyN 求 key1 的差集。
  官方例子:
```sh
key1 = x,a,b,c
key2 = c
key3 = a,d
```

- `SDIFF key1,key2,key3` => x,b
- `SDIFFSTORE dstkey key1 key2 ... keyN` 依据 key2, ..., keyN 求 key1 的差集并存入 dstkey SMEMBERS key 返回某个序列的所有元素
- `SRANDMEMBER key` 随机返回某个序列的元素


# 处理有序集合(sorted sets)的命令 (zsets)

- `ZADD key score member` 添加指定成员到有序集合中，如果目标存在则更新 score(分值，排序用) ZREM key member 从有序集合删除指定成员
- `ZINCRBY key increment member` 如果成员存在则将其增加_increment_，否则将设置一个 score 为 _increment_的成员
- `ZRANGE key start end` 返回升序排序后的指定范围的成员
- `ZREVRANGE key start end` 返回降序排序后的指定范围的成员
- `ZRANGEBYSCORE key min max `返回所有符合 score >= min 和 score <= max 的成员 ZCARD key 返回 有序集合的元素数量 ZSCORE key element 返回指定成员的 SCORE 值 ZREMRANGEBYSCORE key min max 删除符合 score >= min 和 score <= max 条件的所有成员

# 排序(List, Set, Sorted Set)

- `SORT key BY pattern LIMIT start end GET pattern ASC|DESC ALPHA` 按照指定模式排序集合或 List
- `SORT mylist`

## 默认升序 ASC

- `SORT mylist DESC`
- `SORT mylist LIMIT 0 10`
  
## 从序号0开始，取10条
- `SORT mylist LIMIT 0 10 ALPHA DESC`

## 按首字符排序
```sh
SORT mylist BY weight_*
SORT mylist BY weight_* GET object_*
SORT mylist BY weight_* GET object_* GET #
SORT mylist BY weight_* STORE resultkey
```

将返回的结果存放于 resultkey 序列(List)

# 持久控制

- `SAVE` 同步保存数据到磁盘
- `BGSAVE` 异步保存数据到磁盘
- `LASTSAVE` 返回上次成功保存到磁盘的 Unix 时间戳
- `SHUTDOWN` 同步保存到服务器并关闭 Redis 服务器(SAVE+QUIT) BGREWRITEAOF 当日志文件过长时重写日志文件

# 远程控制命令

- `INFO` 提供服务器的信息和统计信息 
- `MONITOR` 实时输出所有收到的请求 
- `SLAVEOF` 修改复制选项

