# 基本数据的性格和能力

类型：字符串，列表，hash，集合，有序集合。

## 通用命令

#### 删除库内表级别的key或者表

> 成功返回1  失败返回0

- del key
- 4.0以后的新特性，后台异步删除，适用于bigkey：UNLINK key

#### 使库内表级别的key或者表过期

> 成功返回1  失败返回0

- 秒级：EXPIRE key 秒

- 毫秒级：PEXPIRE key 毫秒

#### 锁机制

> setnx 没有才允许插入

- set lock 1 ex 过期时间 nx



## 字符串

#### 性格

>  单纯，能容纳字符串和整数和浮点数
>
>  本质字节序列
>
>  动态扩容
>
>  场景：缓存数据，增加查询效率，计数器

#### 能力

- 存储
  - set key value
- 批量存储
  - mset key1 value1 key2 value2 ...
- 获取
  - get key
- 批量获取
  - mget key1 key2 ...
- 数字自增
  - incr key
  - incrby key 步长
- 数字自减
  - decr key
  - decrby key 步长
- 删除
  - del key



## 散列

#### 性格

>  无序字典
>
>  渐进式扩容，不锁表
>
>  数据单位：表或者层级，一个用户对应一张表
>
>  场景：对象存储，分层可以实现一个库内存储不同类型的数据对象

#### 能力

- 存储

  - 4.0之前：hmset 表名 key1 value1 key2 value2
  - hset 表名 key1 value1 key2 value2

- 获取

  - hget 表名 key1

- 批量获取

  - hmget 表名 key1 key2 ...

- 获取所有

  - hgetall 表名

- 统计属性的数量

  - hlen 表名

- 属性值自增

  - hincrby 表名 key 步长 

- 删除表

  - del 表名

- 删除表字段

  > 成功返回1  失败返回0

  - hdel 表名 key



## list

#### 性格

>  双向链表的形成的栈
>
>  只能从左右两端操作, 但是数据下标从左开始从0计数！！！
>
>  场景：最新数据的列表，最新评论等

#### 能力

- 存储

  - 左进：lpush 表名

    > 成功返回列表中数据数量

  - 右进：rpush 表名

    > 成功返回列表中数据数量

- 获取并删除

  - 左出：lpop 表名
  - 右出：rpop 表名

- 只获取不删除，获取某区间的数据

  - lrange 表名 开始下标 结束下标



## 集合sets

#### 性格

>  数据唯一不重复，无序
>
>  场景：去重，获取公共信息，不同信息，所有不重复信息

#### 能力

- 存储

  > 成功返回添加数据的个数

  - sadd 集合 value1 value2 ...

- 获取所有数据

  - smembers 集合

- 删除

  > 成功返回1

  - srem 集合 value

- 随机移除

  - spop 集合

- 交集

  - sinter 集合1 集合2

- 并集

  - sunion 集合1 集合2

- 差集

  - sdiff 集合1 集合2   // 返回集合1中的数据



## 有序集合 sorted sets

#### 性格

>  数据唯一不重复，有权重，所以有序
>
>  内部实现：跳跃表
>
>  场景：去重并根据权重自动排序，各类排行榜

#### 能力

- 存储
  - zadd 集合名 权重 value1 权重 value2 ...
- 获取指定排名区间的值列表
  - 权重默认从低到高排序：zrange 集合名 start end
  - 权重从高到低排序：zrevrange 集合名 start end
- 获取指定权重区间的值
  - 权重默认从低到高排序：zrangebyscore 集合名 start end
  - 权重从高到低排序：zrevrangebyscore 集合名 start end
- 删除
  - zrem 集合 value
- 随机移除
  - spop 集合
- 统计集合内元素个数
  - zcard 集合
- 统计指定权重区间的元素个数
  - zcount 集合 start end
- 获取指定元素的当前排名
  - zrank 集合 value
- 获取指定元素的当前逆向排名
  - zrevrank 集合 value