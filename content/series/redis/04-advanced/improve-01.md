---
title: "优化指南 - 设计"
linktitle: "设计"
toc: true
type: book
date: 2019-03-18T15:04:35+08:00
draft: false
weight: 61
---

## 数据结构

不同数据类型对应的操作的时间复杂度也不一样，选择合适的数据类型可以降低很多时间成本：

### String

| 命令                                                    | 时间复杂度                                          |
| ------------------------------------------------------- | --------------------------------------------------- |
| `SET` key value [EX seconds] [PX milliseconds] [NX\|XX] | O(1)                                                |
| `SETNX` key value                                       | O(1)                                                |
| `SETEX` key seconds value                               | O(1)                                                |
| `PSETEX` key milliseconds value                         | O(1)                                                |
| `GET` key                                               | O(1)                                                |
| `GETSET` key value                                      | O(1)                                                |
| `STRLEN` key                                            | O(1)                                                |
| `APPEND` key value                                      | 平均 O(1)                                           |
| `SETRANGE` key offset value                             | 短字符串平均 O(1)/ 长字符串 O(N)，N 为 `value` 长度 |
| `GETRANGE` key start end                                | O(N)，N 为返回值长度                                |
| `INCR` key                                              | O(1)                                                |
| `INCRBY` key increment                                  | O(1)                                                |
| `INCRBYFLOAT` key increment                             | O(1)                                                |
| `DECR` key                                              | O(1)                                                |
| `DECRBY` key decrement                                  | O(1)                                                |
| `MSET` key value [key value …]                          | O(N)                                                |
| `MSETNX` key value [key value …]                        | O(N)                                                |
| `MGET` key [key …]                                      | O(N)                                                |

### List

| 命令                                    | 时间复杂度                                         |
| --------------------------------------- | -------------------------------------------------- |
| `LPUSH` key value [value …]             | O(1)                                               |
| `LPUSHX` key value                      | O(1)                                               |
| `RPUSH` key value [value …]             | O(1)                                               |
| `RPUSHX` key value                      | O(1)                                               |
| `LPOP` key                              | O(1)                                               |
| `RPOP` key                              | O(1)                                               |
| `RPOPLPUSH` source destination          | O(1)                                               |
| `LREM` key count value                  | O(N)                                               |
| `LLEN` key                              | O(1)                                               |
| `LINDEX` key index                      | O(N)，N 为到达下标 `index` 过程中经过的元素数量    |
| `LINSERT` key BEFORE\|AFTER pivot value | O(N)，N 为寻找 `pivot` 过程中经过的元素数量        |
| `LSET` key index value                  | 头尾元素 O(1)，其他 O(N)                           |
| `LRANGE` key start stop                 | O(S+N)，S 为偏移量 start ， N 为指定区间内元素的数 |
| `LTRIM` key start stop                  | O(N)                                               |
| `BLPOP` key [key …] timeout             | O(1)                                               |
| `BRPOP` key [key …] timeout             | O(1)                                               |
| `BRPOPLPUSH` source destination timeout | O(1)                                               |

### Hash

| 命令                                             | 时间复杂度 |
| ------------------------------------------------ | ---------- |
| `HSET` hash field value                          | O(1)       |
| `HSETNX` hash field value                        | O(1)       |
| `HGET` hash field                                | O(1)       |
| `HEXISTS` hash field                             | O(1)       |
| `HDEL` key field [field …]                       | O(N)       |
| `HLEN` key                                       | O(1)       |
| `HSTRLEN` key field                              | O(1)       |
| `HINCRBY` key field increment                    | O(1)       |
| `HINCRBYFLOAT` key field increment               | O(1)       |
| `HMSET` key field value [field value …]          | O(N)       |
| `HMGET` key field [field …]                      | O(N)       |
| `HKEYS` key                                      | O(N)       |
| `HVALS` key                                      | O(N)       |
| `HGETALL` key                                    | O(N)       |
| `HSCAN` key cursor [MATCH pattern] [COUNT count] | O(1)       |

### Set

| 命令                                             | 时间复杂度                                                   |
| ------------------------------------------------ | ------------------------------------------------------------ |
| `SADD` key member [member …]                     | O(N)                                                         |
| `SISMEMBER` key member                           | O(1)                                                         |
| `SPOP` key                                       | O(1)                                                         |
| `SRANDMEMBER` key [count]                        | 无 conut 时 O(1)，有 conut 时 O(N)                           |
| `SREM` key member [member …]                     | O(N)                                                         |
| `SMOVE` source destination member                | O(1)                                                         |
| `SCARD` key                                      | O(1)                                                         |
| `SMEMBERS` key                                   | O(N)                                                         |
| `SSCAN` key cursor [MATCH pattern] [COUNT count] | O(1)                                                         |
| `SINTER` key [key …]                             | O(N*M)， N 为给定集合当中基数最小的集合， M 为给定集合的个数 |
| `SINTERSTORE` destination key [key …]            | O(N*M)                                                       |
| `SUNION` key [key …]                             | O(N)                                                         |
| `SUNIONSTORE` destination key [key …]            | O(N)                                                         |
| `SDIFF` key [key …]                              | O(N)                                                         |
| `SDIFFSTORE` destination key [key …]             | O(N)                                                         |

### ZSet

| 命令                                                                                                | 时间复杂度                                                                                     |
| --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `ZADD` key score member [[score member] [score member] …]                                           | O(M*log(N))，N 是有序集的基数， M 为成功添加的新成员的数量                                     |
| `ZSCORE` key member                                                                                 | O(1)                                                                                           |
| `ZINCRBY` key increment member                                                                      | O(log(N))                                                                                      |
| `ZCARD` key                                                                                         | O(1)                                                                                           |
| `ZCOUNT` key min max                                                                                | O(log(N))                                                                                      |
| `ZRANGE` key start stop [WITHSCORES]                                                                | O(log(N)+M)，N 为有序集的基数，而 M 为结果集的基数                                             |
| `ZREVRANGE` key start stop [WITHSCORES]                                                             | O(log(N)+M)                                                                                    |
| `ZRANGEBYSCORE` key min max [WITHSCORES] [LIMIT offset count]                                       | O(log(N)+M)                                                                                    |
| `ZREVRANGEBYSCORE` key max min [WITHSCORES] [LIMIT offset count]                                    | O(log(N)+M)                                                                                    |
| `ZRANK` key member                                                                                  | O(log(N))                                                                                      |
| `ZREVRANK` key member                                                                               | O(log(N))                                                                                      |
| `ZREM` key member [member …]                                                                        | O(M*log(N))                                                                                    |
| `ZREMRANGEBYRANK` key start stop                                                                    | O(log(N)+M)                                                                                    |
| `ZREMRANGEBYSCORE` key min max                                                                      | O(log(N)+M)                                                                                    |
| `ZRANGEBYLEX` key min max [LIMIT offset count]                                                      | O(log(N)+M)，N 为有序集合的元素数量， 而 M 则是命令返回的元素数量                              |
| `ZLEXCOUNT` key min max                                                                             | O(log(N))                                                                                      |
| `ZREMRANGEBYLEX` key min max                                                                        | O(log(N)+M)，N 为有序集合的元素数量， 而 M 则为被移除的元素数量                                |
| `ZSCAN` key cursor [MATCH pattern] [COUNT count]                                                    | O(1)                                                                                           |
| `ZUNIONSTORE` destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM\|MIN\|MAX] | O(N)+O(M log(M))，N 为给定有序集基数的总和， M 为结果集的基数                                  |
| `ZINTERSTORE` destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM\|MIN\|MAX] | O(N*K)+O(M*log(M))， N 为给定 key 中基数最小的有序集， K 为给定有序集的数量， M 为结果集的基数 |

### 模拟类型

除了直接使用 redis 数据类型，其他的一些常见数据结构也可以用固定操作模拟出来：

- **堆栈**：lpush + lpop = Stack
- **队列**：lpush + rpop = Queue
- **有限集合**：lpush + ltrim = Capped Collection
- **消息队列**[^1]：lpush + brpop = Message Queue

[^1]: redis5 中提供新的数据结构 `Stream`，直接实现了 Kafka 那种支持多播的消息队列

## 负载

redis 基本的逻辑存储单位是键 (Key) 对象，键底层的编码方式会随着键的类型 / 大小而改变：

| 编码方式 (C 语言实现)                   | 情况                                                                |
| --------------------------------------- | ------------------------------------------------------------------- |
| **`String` 类型**                      |                                                                     |
| int(long 类型整数)                      | long 能存下的整数值                                                 |
| embstr(embstr 类型的简单动态字符串 SDS) | <=32 字节的字符串                                                   |
| raw(简单动态字符串 SDS)                 |                                                                     |
| **`List` 类型**                        |                                                                     |
| ziplist(压缩列表)                       | 列表内元素不超过 512 个并且所有元素长度都小于 64 字节               |
| linkedlist(双端链表)                    |                                                                     |
| **`Hash` 类型**                        |                                                                     |
| ziplist                                 | 哈希内不超过 512 个键值对并且所有键值对的键和值的长度都小于 64 字节 |
| hashtable(字典)                         |                                                                     |
| **`Set` 类型**                         |                                                                     |
| intset(整数集合)                        | 集合内元素不超过 512 个并且所有元素都是整数值                       |
| hashtable                               |                                                                     |
| **`ZSet` 类型**                        |                                                                     |
| ziplist                                 | 有序集合内元素不超过 128 个并且所有元素的长度都小于 64 字节         |
| skiplist(跳跃表和字典)                  |                                                                     |

所以简单的来说，String 长度小于 32 字节，其他复合类型元素个数不超过 512 个 (ZSet 不超过 128 个) 并且元素小于 64 字节或者是整数 (Set) 的时候，是 redis 认为的一个 key 的合理值，超过这个范围 redis 也是允许的，但是关注点就放在存储而不是性能上了。

大键会拖累存储性能。尤其是在时间复杂度不是 O(1)的操作上，性能损失是线性 (eg: `LREM`) 甚至指数 (eg: `ZINTERSTORE`) 上升的。

过小 (零碎) 的键也不合适，它是对性能的一种浪费，比如要存放 ` 用户: 用户信息 `，直接将每个用户存为一个 `String`，相比用 `Hash` 把所有用户存储在一个键上，想要实现 `hgetall` 这样的操作既复杂，效率也更低。

### 集群倾斜 & 热点问题

在集群中，redis 是划分出 16384 个哈希槽，然后将哈希槽平均 (也可以手动指定) 分配到集群节点上。键会通过 `crc16` 算法计算并将结果对 16384 取余，由此将键映射到编号为 0~16383 的哈希槽中。

大键会造成集群倾斜，也就是大键所在节点的内存可能被占满了，而其他节点还空着。极端情况下如果一个键所占空间超过了节点分配的内存，那这个集群可能会永远 `fail` 下去——虽然有大量内存空着，但是没有一个节点能放下这个键了。

大键越多，分布越不均匀，在集群中就越容易出现热点问题 (另一种角度的倾斜)，简单来说，就是

- 由于数据都集中在一个键
- → 对数据的操作都集中在一个键
- → 键位于某个哈希槽
- → 哈希槽所在的节点读写压力非常大
- → 集群其他节点都在划水

假设是 3 个 master 的集群，本来的处理能力可能是 100000 q/s * 3，这样的情况下实际发挥出来的就只有 100000 q/s 了。

针对大键 & 倾斜问题可以有以下措施：

1. 将可能的大键进行拆分，比如将一个大 List 拆成 List0~List9；
2. 在集群配置中开启 `readonly` 以降低主节点读压力 (详见 [《Redis 集群相关命令》](cluster_command.md))；
3. 根据实际情况，修改 redis 变更编码类型的阈值，比如设定 `list-max-ziplist-entries=1024` 让元素在 1024 以内的列表都用 ziplist 编码；
