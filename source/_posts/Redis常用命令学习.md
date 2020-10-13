---
title: Redis常用命令学习
date: 2019-04-27 14:24:00
tags: Redis
categories: Redis
---

 学习Redis常用命令

<!--more-->

## 各个结构常用命令

### String

* SET
* GET
* DEL(适用于所有类型)

### 列表

* RPUSH
* LRANGE key 0 -1
* LINDEX
* LPOP

### 集合

* SADD
* SMEMBERS
* SISMEMBER
* SREM

### 散列

## String

### SET

> set key value \[expiration EX seconds|PX milliseconds][NX|XX]

将字符串值 `value` 关联到 `key` 。如果 `key` 已经持有其他值， `SET` 就覆写旧值， 无视类型。当 `SET` 命令对一个带有生存时间（TTL）的键进行设置之后， 该键原有的 TTL 将被清除。

从 Redis 2.6.12 版本开始， `SET` 命令只在设置操作成功完成时才返回 `OK` ； 如果命令使用了 `NX`或者 `XX` 选项， 但是因为条件没达到而造成设置操作未执行， 那么命令将返回空批量回复（NULL Bulk Reply）。

> SET key “value”
> GET key
>
> SET 	key “value” EX seconds：将键的过期时间设置为 `seconds` 秒。 执行 `SET key value EX seconds` 的效果等同于执行 `SETEX key seconds value` 。
>
> SET key “value” NX：只在键不存在时， 才对键进行设置操作。 执行 `SET key value NX` 的效果等同于执行 `SETNX key value` 。
>
> SET key “value” XX：只在键已经存在时， 才对键进行设置操作。

### GET

返回与键 `key` 相关联的字符串值。

### GETSET

将键 `key` 的值设为 `value` ， 并返回键 `key` 在被设置之前的旧值。 键 `key` 在被设置之前并不存在， 那么命令返回 `nil` 。

### STRLEN key

返回键 `key` 储存的字符串值的长度。当键 `key` 不存在时， 命令返回 `0` 。

### APPEND

如果键 `key` 已经存在并且它的值是一个字符串， `APPEND` 命令将把 `value` 追加到键 `key` 现有值的末尾。如果 `key` 不存在， `APPEND` 就简单地将键 `key` 的值设为 `value` ， 就像执行 `SET key value` 一样。

### SETRANGE

>  SETRANGE key offset value

从偏移量 `offset` 开始， 用 `value` 参数覆写(overwrite)键 `key` 储存的字符串值。不存在的键 `key` 当作空白字符串处理。

### GETRANGE

>  GETRANGE key start end

返回键 `key` 储存的字符串值的指定部分， 字符串的截取范围由 `start` 和 `end` 两个偏移量决定 (包括 `start` 和 `end` 在内)。

### INCR

为键 `key` 储存的数字值加上一。如果键 `key` 不存在， 那么它的值会先被初始化为 `0` ， 然后再执行 `INCR` 命令。如果键 `key` 储存的值不能被解释为数字， 那么 `INCR` 命令将返回一个错误。

### INCRBY

> INCRBY key increment

为键 `key` 储存的数字值加上增量 `increment` 。如果键 `key` 不存在， 那么键 `key` 的值会先被初始化为 `0` ， 然后再执行 `INCRBY` 命令。

## List

### LPUSH

> LPUSH key value

将一个或多个值 `value` 插入到列表 `key` 的表头。如果有多个 `value` 值，那么各个 `value` 值按从左到右的顺序依次插入到表头： 比如说，对空列表 `mylist` 执行命令 `LPUSH mylist a b c` ，列表的值将是 `c b a` ，这等同于原子性地执行 `LPUSH mylist a` 、 `LPUSH mylist b` 和 `LPUSH mylist c` 三个命令。

### LPUSHX

> LPUSHX key value

将值 `value` 插入到列表 `key` 的表头，当且仅当 `key` 存在并且是一个列表。

和 LPUSH key value [value …\] 命令相反，当 `key` 不存在时， [LPUSHX](http://redisdoc.com/list/lpushx.html#lpushx) 命令什么也不做。

### RPUSH

> RPUSH key value [value …]

LPUSh和RPUSH都会返回当前列表的长度

### LPOP

> LPOP key

移除并返回列表 key的头元素

### RPOP

### RPOPLPUSH

> RPOPLPUSH source destination

执行两个动作：1. 将列表source中的最后一个元素弹出并返回给客户端 2. source弹出的元素插入destination，作为destination的头元素。

如果 `source` 和 `destination` 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转(rotation)操作。

### LREM

> LREM key count value

根据count 的值，移除列表中与 value相等的元素  

* `		count > 0` : 从表头开始向表尾搜索，移除与 `value` 相等的元素，数量为 `count` 。
* `count < 0` : 从表尾开始向表头搜索，移除与 `value` 相等的元素，数量为 `count` 的绝对值。` 
* `count = 0` : 移除表中所有与 `value` 相等的值。

### LLEN

> LLEN key

返回key的长度

### LINDEX key index

返回列表中，下表为index的元素。0表示第一个元素，-1表示倒数第一个，-2 表示倒数第二个。越界返回nil.

### LINSERT key BEFORE|AFTER pivot value

将值 `value` 插入到列表 `key` 当中，位于值 `pivot` 之前或之后。当 `pivot` 不存在于列表 `key` 时，不执行任何操作。当 `key` 不存在时， `key` 被视为空列表，不执行任何操作。

### LSET key index value

将列表 `key` 下标为 `index` 的元素的值设置为 `value` 

### LRANGE key start stop

返回列表 `key` 中指定区间内的元素，区间以偏移量 `start` 和 `stop` 指定。

 `stop` 下标也在 [LRANGE](http://redisdoc.com/list/lrange.html#lrange) 命令的取值范围之内(闭区间)

超出范围的下标值不会引起错误。

如果 `start` 下标比列表的最大下标 `end` ( `LLEN list` 减去 `1` )还要大，那么 [LRANGE](http://redisdoc.com/list/lrange.html#lrange) 返回一个空列表。

如果 `stop` 下标比 `end` 下标还要大，Redis将 `stop` 的值设置为 `end` 。

查看key中的所有元素：`LRANGE key 0 -1 `

### LTRIM key start stop

对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

> LRANGE fruit 0 -1

## Hash

### HSET hash field value

将哈希表 `hash` 中域 `field` 的值设置为 `value` 。

如果给定的哈希表并不存在， 那么一个新的哈希表将被创建并执行 `HSET` 操作。

如果域 `field` 已经存在于哈希表中， 那么它的旧值将被新值 `value` 覆盖。

### HSETNX hash field value

当且仅当域 `field` 尚未存在于哈希表的情况下， 将它的值设置为 `value` 。

如果给定域已经存在于哈希表当中， 那么命令将放弃执行设置操作。

如果哈希表 `hash` 不存在， 那么一个新的哈希表将被创建并执行 `HSETNX` 命令。

### HGET hash field

返回哈希表中给定域的值

### HEXISTS hash field

检查field是否存在于hash中，存在返回1，否则0

### HDEL key field [field …]

删除

### HLEN key

### HSTRLEN key field

返回哈希表 `key` 中， 与给定域 `field` 相关联的值的字符串长度（string length）。

### HINCRBY key field increment

哈希表 `key` 中的域 `field` 的值加上增量 `increment` 。

增量也可以为负数，相当于对给定域进行减法操作。

如果 `key` 不存在，一个新的哈希表被创建并执行 [HINCRBY](http://redisdoc.com/hash/hincrby.html#hincrby) 命令。

如果域 `field` 不存在，那么在执行命令前，域的值被初始化为 `0` 。

### HINCRBYFLOAT key field increment

为哈希表 `key` 中的域 `field` 加上浮点数增量 `increment` 。

### HMSET key field value [field value …]

同时将多个field-value 设置到哈希表中

### HMGET key field [field …]

返回哈希表中，一个或多个给定的值

### HKEYS key

返回哈希表 `key` 中的所有域。

### HVALS key

返回哈希表 `key` 中所有域的值。

### HGETALL key

返回哈希表中，所有的域和值，以列表形式。在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

## Sets

集合和列表都能存储多个字符串，不同之处在于，列表可存储多个相同的字符串，而集合通过散列表保证自己存储的每个字符串各不相同。

### **SADD** key member [member …]

将一个或多个 `member` 元素加入到集合 `key` 当中，已经存在于集合的 `member` 元素将被忽略。

### SISMEMBER key member

判断 `member` 元素是否集合 `key` 的成员。是，返回1，不是或key不存在，0.

### SPOP key

移除并返回集合中的一个随机元素。

### SRANDMEMBER key [count]

如果命令执行时，只提供了 `key` 参数，那么返回集合中的一个随机元素。

* 如果 `count` 为正数，且小于集合基数，那么命令返回一个包含 `count` 个元素的数组，数组中的元素**各不相同**。如果 `count` 大于等于集合基数，那么返回整个集合。
* 如果 `count` 为负数，那么命令返回一个数组，数组中的元素**可能会重复出现多次**，而数组的长度为 `count` 的绝对值。

### SREM key member

移除key中的一个或多个元素，不存在的将被忽略

### SMOVE source destination member

将 `member` 元素从 `source` 集合移动到 `destination` 集合。

[SMOVE](http://redisdoc.com/set/smove.html#smove) 是原子性操作。

### SCARD key

返回集合元素数量。key不存在，返回0.

### **SMEMBERS** key

返回集合 `key` 中的所有成员。不存在的 `key` 被视为空集合。

### SINTER key [key …]

返回所有集合的交集成员。

### SINTERSTORE destination key [key …]

这个命令类似于 [SINTER key [key …\]](http://redisdoc.com/set/sinter.html#sinter) 命令，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

`destination` 可以是 `key` 本身。

### SUNION key [key …]

返回一个集合的全部成员，该集合是所有给定集合的并集。

### SUNIONSTORE destination key [key …]

### SDIFF

返回多个集合的差集。返回存在于第一个集合而不在其他集合中的元素

### SDIFFSTORE destination key [key …]

这个命令的作用和 [SDIFF key [key …\]](http://redisdoc.com/set/sdiff.html#sdiff) 类似，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

## Sorted Sets

### ZADD key score member [[score member][score member] …]

将一个或多个 `member` 元素及其 `score` 值加入到有序集 `key` 当中。

如果某个 `member` 已经是有序集的成员，那么更新这个 `member` 的 `score` 值，并通过重新插入这个 `member` 元素，来保证该 `member` 在正确的位置上。

`score` 值可以是整数值或双精度浮点数。

### ZSCORE key member

返回有序集 `key` 中，成员 `member` 的 `score` 值。

### ZINCRBY key increment member

为有序集 `key` 的成员 `member` 的 `score` 值加上增量 `increment` 。

可以通过传递一个负数值 `increment` ，让 `score` 减去相应的值，比如 `ZINCRBY key -5 member` ，就是让 `member` 的 `score` 值减去 `5` 。

当 `key` 不存在，或 `member` 不是 `key` 的成员时， `ZINCRBY key increment member` 等同于 `ZADD key increment member` 。

### ZCARD key

返回key的长度

### ZCOUNT key min max

返回key中score在[min … max]中成员的数量

### ZRANGE key start stop [WITHSCORES]

返回key中，指定区间内的成员，成员位置按照score值递增排序；相同score的成员按照字典顺序。

如果你需要成员按 `score` 值递减(从大到小)来排列，请使用 ZREVRANGE key start stop [WITHSCORES\]](http://redisdoc.com/sorted_set/zrevrange.html#zrevrange) 命令。

### ZREVRANGE key start stop [WITHSCORES]

和ZRANGE 类似，只不过排序规则变为递减。

### ZRANGEBYSCORE key min max [WITHSCORES\][LIMIT offset count\]

返回有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员。有序集成员按 `score` 值递增(从小到大)次序排列。

### ZREVRANGEBYSCORE key min max [WITHSCORES\][LIMIT offset count\]

### ZRANK

ZRANK key member

返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递增(从小到大)顺序排列。排名以 `0` 为底，也就是说， `score` 值最小的成员排名为 `0` 。

### ZREVRANK

> ZREVRANK key member

逆序

### ZREM

> ZREM key member

移除。

返回移除成功的数量。

### ZREMRANGEBYRANK

> ZREMRANGEBYRANK key start stop

移除有序集 `key` 中，指定排名(rank)区间内的所有成员。

### ZREMRANGEBYSCORE

> ZREMRANGEBYSCORE key min max

移除score在[min … max]之间的所有成员

### ZRANGEBYLEX

> ZRANGEBYLEX key min max [LIMIT offset count]

返回给定的有序集合键 `key` 中， 值介于 `min` 和 `max` 之间的成员。

合法的 `min` 和 `max` 参数必须包含 `(` 或者 `[` ， 其中 `(` 表示开区间（指定的值不会被包含在范围之内）， 而 `[` 则表示闭区间（指定的值会被包含在范围之内）。

> ZRANGEBYLEX myset [a [d

返回member中值在 a和d之间（包含）的成员。

特殊值 `+` 和 `-` 在 `min` 参数以及 `max` 参数中具有特殊的意义， 其中 `+` 表示正无限， 而 `-` 表示负无限。 因此， 向一个所有成员的分值都相同的有序集合发送命令 `ZRANGEBYLEX <zset> - +` ， 命令将返回有序集合中的所有元素。

### ZLEXCOUNT

> ZLEXCOUNT key min max

与ZRANGEBYLEX类似，不过是返回数量。

### ZREMRANGEBYLEX

> ZREMRANGEBYLEX key min max

对于一个所有成员的分值都相同的有序集合键 `key` 来说， 这个命令会移除该集合中， 成员介于 `min` 和 `max` 范围内的所有元素。

## DB

### EXISTS  key

检查key是否存在

### TYPE key

key的存储值的类型

### RENAME key newkey

key改名为newkey

### RENAMENX

### MOVE key db

将当前数据库的 `key` 移动到给定的数据库 `db` 当中。

**redis默认使用数据库 0**

### DEL key [key …]

删除一个或多个key

### RANDOMKEY

从当前数据库随机返回一个key

### DBSIZE

返回当前数据库的key 的数量。

### KEYS pattern

查找所有符合给定模式 `pattern` 的 `key` ， 比如说：

* `KEYS *` 匹配数据库中所有 `key` 。
* `KEYS h?llo` 匹配 `hello` ， `hallo` 和 `hxllo` 等。
* `KEYS h*llo` 匹配 `hllo` 和 `heeeeello` 等。
* `KEYS h[ae]llo` 匹配 `hello` 和 `hallo` ，但不匹配 `hillo` 。

### SCAN

[redisdoc - SCNA](<http://redisdoc.com/database/scan.html>)

### SORT

[redisdoc - sort](<http://redisdoc.com/database/sort.html>)

### FLUSHDB

清空当前数据库。总是返回OK

### FLUSHALL

清空整个Redis

### SELECT index

切换到指定数据库

### OBJECT ENCODING

查看一个数据库键的值对象编码。

## 自动过期

### EXPIRE

> EXPIRE key seconds

为给定 `key` 设置生存时间(单位：秒)，当 `key` 过期时(生存时间为 `0` )，它会被自动删除。

可以对一个已经带有生存时间的 `key` 执行 `EXPIRE` 命令，新指定的生存时间会取代旧的生存时间。

### PEXPIRE

> PEXPIRE key milliseconds

毫秒为单位

### EXPIREAT

> EXPIREAT key timestamp

秒数时间戳

### PEXPIREAT

> PEXPIREAT key milliseconds-timestamp

### PERSIST 

> PERSIST key

移除key 的生存时间

### TTL key

以秒为单位返回剩余生存时间

## 持久化

### SAVE

[SAVE](http://redisdoc.com/persistence/save.html#save) 命令执行一个同步保存操作，将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘。

SAVE会阻塞所有客户端。

### BGSAVE

在后台异步(Asynchronously)保存当前数据库的数据到磁盘。

[BGSAVE](http://redisdoc.com/persistence/bgsave.html#bgsave) 命令执行之后立即返回 `OK` ，然后 Redis fork 出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。

### BGREWRITEAOF

执行一个 [AOF文件](http://redis.io/topics/persistence#append-only-file) 重写操作。重写会创建一个当前 AOF 文件的体积优化版本。

## 发布与订阅

### PUBLISH

> PUBLISH channel message

将信息 `message` 发送到指定的频道 `channel` 。返回接收到消息 message 的订阅者数量

### SUBSCRIBE

> SUBSCRIBE channel [channel …]

订阅给定的一个或多个频道的信息。

## 事务

### MULTI

标记一个事务块的开始。

事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 [EXEC](http://redisdoc.com/transaction/exec.html#exec) 命令原子性(atomic)地执行。

### EXEC

执行所有事务块内的命令。

### DISCARD

取消事务

### WATCH

监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。



## 参考

* [redisdoc](<http://redisdoc.com/>)
* [redis-commands](<https://redis.io/commands>)