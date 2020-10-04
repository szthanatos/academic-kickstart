---
title: "优化指南 - 运维"
linktitle: "运维"
toc: true
type: book
date: 2019-03-18T15:04:35+08:00
draft: false
---

## 监控

为了发现前面所说的问题，需要开发 / 运维人员不断的监控 redis 运行情况。

### redis-cli 查询

部分信息无法通过 redis 命令直接获取，但是可以通过 `redis-cli [参数]` 获取：

`–-bigkeys`

后台 scan 出每种数据类型中较大的 key

`--latency`

服务端响应延时

### slowlog 命令

在客户端执行 `slowlog get [n]` 可以获取最慢的 n 条执行命令的记录

### info 命令

返回服务器信息，性能监测的时候注意其中的几个部分：

**memory**：`mem_fragmentation_ratio`

内存碎片率，`used_memory_rss`(系统分配内存总量) 和 `used_memory`(Redis 分配器分配的内存总量) 的比值。

在 1-1.5 之间都是合理值，<1 则说明内存已经占满，正在和硬盘进行内存交换，性能下降严重，>1.5 则说明碎片过多需要清理了。

**stats**：`latest_fork_usec`

最近一次 fork 操作耗时

**persistence**：`aof_delayed_fsync`

被延迟的 fsync 调用数量

**clients**：`connected_clients`，`blocked_clients`

已连接客户端的数量和正在等待阻塞命令的客户端的数量

### monitor 命令

可以用来监测一个节点一段时间内执行的命令，从而统计出热点 key。但是 monitor 自己也是有内存占用的，所以不能频繁、持续的使用。

## 部署

### 网络

影响 redis 性能的最主要因素是网络。

按官方基准测试来说，对于 10kb 以内的数据，redis 的处理能力在 100000q/s 以上。

那么假设每次 set/get 的 4kb 大小的字符串，这时占用的带宽就有 3.2 Gbit/s ，千兆网卡 (1 Gbit/s) 就不够用了，得换万兆网卡 (10 Gbit/s) 才能满足需求，可见想跑满 redis 的 CPU 计算力对网络的要求是很夸张的。

当然，这个例子比较极端，redis 官方推荐的网络环境下每次传输的包最好不超过一个 `MTU`(大约 1500 bytes)。

如果完全抛开网络因素，客户端服务端都在单机上时，使用 Unix 域套接字 (`Unix domain sockets`，也叫 `IPC(inter-precess communication) socket` 进程间通信套接字) 替换默认的 TCP/IP 连接方式，能额外再有 50% 的吞吐量提升(不过在大量使用 pipeline 的情况下就没差这么多了)。

启用 Unix 域套接字需要在配置文件中取消注释：

```bash
# unixsocket 路径
unixsocket /tmp/redis.sock

# unixsocket 权限
unixsocketperm 700
```

之后就可以在客户端使用指定方式连接了，以 python 客户端为例：

```python
import redis

redis_connect = redis.Redis(unix_socket_path='/tmp/redis.sock')
pass
```

### CPU

redis 更倾向于具有更大缓存而不是更多核的 CPU，在多核的情况下，redis 性能会受 NUMA 配置和进程所处位置的影响，指定客户端和服务器使用同一 CPU 的两个不同核心可以使从 L3 缓存获得的收益最大化。

另外，redis 在 Inter 和 AMD 的 CPU 上的表现也有差别，在某些情况下在 AMD 的 CPU 上性能可能只有 Inter 的一半。

### 内存

只有在面对大于 10KB 的数据的时候，内存频率 / 带宽才会影响 redis 性能，所以一般不用去考虑。内存大小只会影响能存放的数据量。

### 连接数

redis 可以在 60000 多个连接时维持 50000 q/s 的性能，但是根据官方测试，具有 30000 个连接的 redis 实例只能处理 100 个连接实例可实现的吞吐量的一半。

### 虚拟化

虚拟机中的 redis 性能肯定是低于实机上的，系统调用和中断上面浪费的太多。
