---
title: Redis常用命令
date: 2017-11-20
categories:
- 后端
- Redis
tags:
- 后端
- Redis
---
## String
命令|解释
--|--
INCR key| 增加值+1
DECR key| 减少值-1
MSET|一次set多个值
MGET|一次get多个值
INCRBY key|按参数增加值
DECRBY key|按参数减少值
APPEND key value|拼接字符串
GETRANGE key offset|截取字符串
SETRANGE key offset value|按位置更改字符串
STRLEN key|获取键对应的值的长度
SETBIT key offset value|设置或者清空key的value(字符串)在offset处的bit值
GETBIT key offset| 获得key的value(字符串)在offset处的bit值
BITCOUNT key start end| 统计字符串的二级制码中，有多少个'1'
## List
命令|解释
--|--
RPUSH(LPUSH) key value| 从右端向key插入值 
LRANGE key start stop|按范围返回值
LINDEX key index| 按索引返回值
RPOP(LPOP)|从右侧弹出一个值
LTRIM key start stop| 按范围从左侧删除值
BRPOP(BLPOP) key [key ...] timeout|阻塞式从右侧弹出值，有多个key时，从key的左边向右边遍历弹出值，直到没有值可以被弹出
RPOPLPUSH source destination| 原子性地返回并移除存储在 source 的列表的最后一个元素（列表尾部元素）， 并把该元素放入存储在 destination 的列表的第一个元素位置（列表头部）
<!-- more -->
## Set
命令|解释
--|--
SADD key member [member...] | 添加一个或多个元素
SMEMBERS key|获得key的所有值
SISMEMBER key member|判断一个member在不在set中
SREM key member [member...]| 删除一个或多个member
SCARD key| 获取集合中元素数量
SPOP key [count] | 随机弹出一个或多个元素
SRANDMEMBER key [count]| 返回一个或多个元素
SMOVE source destination member| 把一个元素从一个set移动到另一个set
SDIFF| 求两个set的差集
SDIFFSTORE| 求两个set的差集并保存
SINTER|求两个set的交集
SINTERSTORE|求两个set的交集并保存
SUNION|求两个set的并集
SUNIONSTORE|求两个set的并集并保存
## Hash
命令|解释
--|--
HSET key field value| 添加一个hash
HDEL key field [field ...]| 删除一个或多个键值
HGET key field| 获得key下面field键的值
HGETALL key| 返回key下面所有的键值对
HKEYS key| 返回key下面所有的键
HVALS key|返回key下面所有的值
HINCRBY key field increment| key下面field的值增加1
HLEN key| 返回key下面值键值对的数量
HMSET key field value [field value...] | 添加多个hash
HEXISTS key field| 判断一个key是否在hash内
## Zset
命令|解释
--|--
ZRANGE key start stop [WITHSCORES]| 根据指定的index返回，返回sorted set的成员列表
ZRANGEBYSCORE key min max [WITHSCORES]|返回有序集合中指定分数区间内的成员，分数由低到高排序
ZREM key member [member...]| 从排序的集合中删除一个或多个成员
ZINCRBY key increment member| 增加一名成员的评分
ZCOUNT key min max| 返回分数范围内的成员数量
ZRANK key member| 返回成员在集合中的索引
ZSCORE key member| 返回成员的得分
ZREVRANK key member|返回分数从高到低的成员的排名
ZREVRANGE key start stop [WITHSCORES]|返回成员的范围，分数从高到低排序
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]|按照分数从大到小返回范围元素，可以增加limit进行数量限制
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]|按照分数从小到大返回范围元素，可以增加limit进行数量限制
ZADD key [NX or XX] [CH] [INCR] score member [score member ...]|将所有指定成员添加到键为key有序集合（sorted set）里面
ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [SUM or MIN orMAX]|计算交集，并把结果放到destination中
ZUNIONSCORE|计算并集，并把结果放到destination中
ZRANGEBYLEX key min max [LIMIT offset count]|返回指定成员区间内的成员，按字典正序排列
ZREM key member [member ...]| 删除一个或多个成员