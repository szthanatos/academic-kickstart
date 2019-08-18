---
title: "Redis集群相关命令"
linktitle: "集群相关命令"
toc: true
type: docs
date: 2018-08-30T21:26:18+08:00
draft: false
menu:
  redis:
    weight: 2

weight: 2
---

## 集群信息

### 节点信息

集群节点相关信息可以通过`cluster nodes`命令获取：

```bash
127.0.0.1:6379> cluster nodes
5022fa41642195d74200fd512c08653dc12609e7 172.17.0.2:6380@16380 master - 0 1552814467000 2 connected 5461-10922
730d05ee4da3147d0885c6d47437465c94409f74 172.17.0.2:6383@16383 slave 25a2a1d8c96e9473d4cb3c8b0077d3b7b07dd5c0 0 1552814468555 5 connected
25a2a1d8c96e9473d4cb3c8b0077d3b7b07dd5c0 172.17.0.2:6379@16379 myself,master - 0 1552814468000 1 connected 0-5460
cf0be45c6a05c0d332b7356a7f57de95b32c3a71 172.17.0.2:6384@16384 slave 5022fa41642195d74200fd512c08653dc12609e7 0 1552814468654 6 connected
55d264b148ce35903928964ac017d682fc803eab 172.17.0.2:6381@16381 master - 0 1552814468000 3 connected 10923-16383
1a60baf4c2254b2e3a37cf6215d42b316ffdccc7 172.17.0.2:6382@16382 slave 55d264b148ce35903928964ac017d682fc803eab 0 1552814468000 4 connected
127.0.0.1:6379>
```

可以看到，每一行即是一个节点的信息，以第一行为例，每个字段的含义分别是：

| 字段                  | 含义                                                                      |
| --------------------- | ------------------------------------------------------------------------- |
| 502......9e7          | node_id，节点标识                                                         |
| 172.17.0.2:6380@16380 | IP端口                                                                    |
| master                | 身份，一般就是`myself/master/slave`，其他`fail?/handshake/noaddr/noflags` |
| -                     | 对应主节点的node_id(如果你是从节点的话，否则就是`-`)                      |
| 0                     | `0`表示没有待发送的`ping`，否则是要发送`ping`的时间戳                     |
| 1552814467000         | 收到上一个`pong`命令的时间戳                                              |
| 2                     | 权重                                                                      |
| connected             | 状态，`connected`or`disconnected`                                         |
| 5461-10922            | 哈希槽，连续的哈希槽用`-`连接，离散的用`,`隔开                            |

### 状态信息

集群信息通过`cluster info`获取：

```bash
127.0.0.1:6379> cluster info
# 集群状态
cluster_state:ok
# 已分配的哈希槽数量，不是16384就有问题了
cluster_slots_assigned:16384
# 正确的哈希槽数量，如果有哈希槽分配到了离线的节点上就不是这个数字了
cluster_slots_ok:16384
# 临时错误哈希槽数，比如网络波动但节点还是正常的，那就是pfail，节点确认离线了就是fail
cluster_slots_pfail:0
cluster_slots_fail:0
# 集群节点数
cluster_known_nodes:6
# 集群规模
cluster_size:3
# 当前最大权重
cluster_current_epoch:6
# 本节点的权重
cluster_my_epoch:1
# 集群建立至今发送/接受的ping/pong/meet消息数
cluster_stats_messages_ping_sent:1554
cluster_stats_messages_pong_sent:1558
cluster_stats_messages_sent:3112
cluster_stats_messages_ping_received:1553
cluster_stats_messages_pong_received:1554
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:3112
127.0.0.1:6379>
```

## 集群操作

就不挨个详述了，写了个速查表如下，全部命令详见[redis官方文档](https://redis.io/commands)。

| 命令                                         | 含义                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| **信息**                                     |                                                              |
| `cluster info`                               | 返回集群基本信息                                             |
| `cluster nodes`                              | 返回全部节点                                                 |
| **控制**                                     |                                                              |
| `cluster count-failure-reports <node_id>`    | 返回节点的错误报告数                                         |
| `cluster failover [FORCE/TAKEOVER]`          | 在从节点上执行，测试故障转移                                 |
| `cluster set-config-epoch <config-epoch>`    | 为新节点设置权重                                             |
| `cluster saveconfig`                         | 将节点的配置文件保存到硬盘里面                               |
| `cluster reset [HARD/SOFT]`                  | 重置没有key的当前节点(所以应该先flushall)                    |
| `cluster readonly`                           | 将从节点置为可读(默认情况下，指向从节点的连接会被转向主节点) |
| `cluster readwrite`                          | 将从节点置为读写(相当于还原为`readonly`之前的状态)           |
| **节点**                                     |                                                              |
| `cluster meet <ip> <port>`                   | 将节点加入集群作为主节点                                     |
| `cluster replicate <node_id>`                | 将当前节点作为<node_id>的从节点加入集群                      |
| `cluster replicas <node_id>`                 | redis5.0开始支持，列出指定主节点的从节点                     |
| `cluster slaves <node_id>`                   | 不再建议被使用，基本同replicas                               |
| `cluster replicaof <ip> <port>`              | redis5.0开始支持，将当前节点置为指定节点的从节点             |
| `cluster slaveof <ip> <port>`                | 不再建议被使用，基本同replicaof                              |
| `cluster forget <node_id>`                   | 从集群中移除<node_id>节点（前提是没给他分配哈希槽/内容）     |
| **哈希槽**                                   |                                                              |
| `cluster addslots <slot>,...`                | 将一个或多个哈希槽添加到当前节点                             |
| `cluster delslots  <slot>,...`               | 从当前节点移除哈希槽                                         |
| `cluster flushslots`                         | 清空当前节点的哈希槽                                         |
| `cluster setslot <slot> node <node_id>`      | 将**未分配的**哈希槽分配给指定节点                           |
| `cluster setslot <slot> migrating <node_id>` | 将本节点的哈希槽合并到指定节点                               |
| `cluster setslot <slot> importing <node_id>` | 从指定节点引入哈希槽到本节点                                 |
| `cluster setslot <slot> stable`              | 取消哈希槽的移动，_主要是修复`redis-trib fix`引发的问题←_←_  |
| `cluster flushslots`                         | 清空当前节点的哈希槽                                         |
| `cluster flushslots`                         | 清空当前节点的哈希槽                                         |
| **键**                                       |                                                              |
| `cluster keyslot <key>`                      | 计算key应该被放在哪个槽                                      |
| `cluster countkeysinslot <slot>`             | 返回哈希槽中key的数量                                        |
| `cluster getkeysinslot <slot> <count>`       | 从哈希槽中返回指定数量个key                                  |

{{% alert note %}}
**ps1**：从redis5.0起，`slave`关键字将被`replicate`取代，相关命令也会逐步被替代
{{% /alert %}}

{{% alert note %}}
**ps2**：`readonly`这个额外说一下，默认情况下，redis的从节点不处理客户端请求，只负责同步主节点的数据，设置`readonly`之后，指向从节点的读请求会被执行，这样相当于为主节点分担了读的压力(也算是scale了)，但是还是存在两个限制：

1. 写请求还是会转移给主节点；
2. 要读的key的哈希槽不属于这个节点，请求一样会被转移；
{{% /alert %}}
