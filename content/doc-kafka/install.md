---
title: Kafka -01- 安装配置
linktitle: 安装配置
toc: true
type: book
date: 2018-12-06T10:09:46+08:00
draft: false
weight: 10
---

## 环境说明

| 当前版本 | 发布日期   | 下载地址                                                                                      |
| -------- | ---------- | --------------------------------------------------------------------------------------------- |
| 2.0      | 2018-07-30 | [官方 2.0.0 镜像](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.0.0/kafka_2.11-2.0.0.tgz) |

## 安装步骤

_** 注意：** 文中以 `{}` 包裹起来的内容需要自己替换，并非直接使用_

### 0. 环境准备

| 基础环境              | 说明                                                                                                                                                                                |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Java**              | Java 版本应该在 8(jdk1.8) 或以上，以更好的支持 G1 回收                                                                                                                                   |
| **硬件参数**          | CPU: 英特尔至强 E5-2650 v4 *2 (共计 24 核) <br>Mem: DDR4 内存 - 32GB* 8 <br>Sto: 2000GB * 8 raid 0                                                                                        |
| **文件路径**          | /home/tools/kafka_2.11-2.0.0/                                                                                                                                                       |
| **数据存放位置**      | /home/sdb/kafka-logs,/home/sdc/kafka-logs, <br>/home/sdd/kafka-logs,/home/sde/kafka-logs, <br>/home/sdf/kafka-logs,/home/sdg/kafka-logs, <br>/home/sdh/kafka-logs,/home/sdi/kafka-logs |
| **zookeeper 集群位置** | 10.10.20.83:2181,10.10.20.84:2181,10.10.20.85:2181                                                                                                                                  |

### 1. 下载安装

下载最新版本 Kafka，解压到指定目录，无需其他操作即完成安装。

```bash
tar -xzf kafka_2.11-2.0.0.tgz -C /home/tools
cd kafka_2.11-2.0.0
```

### 2. 配置集群参数

修改 `config/server.properties`

基本参数如下：

```bash
# broker 唯一 id，值为不重复正整数
broker.id={n: int}

# 服务监听地址
listeners=PLAINTEXT://{your.host}:9092

# 日志存放位置列表，以逗号隔开
log.dirs={data.storage.list}

# zookeeper 地址列表，以逗号隔开
zookeeper.connect={zookeeper.server.list}
```

优化参数配置如下：

```bash
# 消息处理线程数，建议数量为 cpu 核数加 1
num.network.threads=25

# 磁盘 IO 的线程数, 建议为 cpu 核数 2 倍，最大不超过 3 倍
num.io.threads=48

# 拉取线程数，影响 follower 的 I/O 并发度，单位时间内 leader 持有更多请求，相应负载会增大
num.replica.fetchers=2

# 分区数量配置，根据业务情况修改
num.partitions=16

# 消息日志备份数，默认是 1
default.replication.factor=2

# 刷盘 (写入文件到磁盘) 间隔消息数，建议设为 10000
log.flush.interval.messages=10000

# 刷盘间隔毫秒数，建议 1 秒 (1000)
log.flush.interval.ms=1000

# 日志保留小时数
log.retention.hours=48

# 段文件大小，过小会产生很多小文件降低性能，过大会影响快速回收磁盘空间以及 Kafka 重启后的载入速度
og.segment.bytes=1073741824

# 最大字节数，默认 1M，可以调到 5M 以上
replica.fetch.max.bytes=5242880

# 可接受数据大小，受限于 java int 类型的取值范围, 超出后会报 OOM 异常
socket.request.max.bytes=2147483600

# 日志传输时候的压缩格式，可选择 lz4, snappy, gzip, 不压缩。建议打开压缩，可以提高传输性能
compression.type=snappy

# 是否允许通过管理工具删除 topic，默认是 false
delete.topic.enable=true

# 是否允许程序自动创建 Topic
auto.create.topics.enable=false
```

### 3. 配置日志参数

修改 `config/log4j.properties` 的 jog4j 参数，提高 Kafka 操作日志（和数据日志区分）的日志级别，以降低日志输出相关资源占用。具体可更改配置如下：

```bash
# Kafka2.0 默认只有 controller 是 TRACE 级别，可以改为 INFO，其他 INFO 级别可以适当提升为 WARN

# zookeeper 日志级别，
log4j.logger.org.I0Itec.zkclient.ZkClient=INFO
log4j.logger.org.apache.zookeeper=INFO

# 主日志级别
log4j.logger.kafka=INFO
log4j.logger.org.apache.kafka=INFO

# request 日志级别，只有当需要调试时才有必要输出
log4j.logger.kafka.request.logger=WARN, requestAppender
log4j.additivity.kafka.request.logger=false

# 需要调试时解除以下三行注释，并将 RequestChannel$ 设为 TRACE
# log4j.logger.kafka.network.Processor=TRACE, requestAppender
# log4j.logger.kafka.server.KafkaApis=TRACE, requestAppender
# log4j.additivity.kafka.server.KafkaApis=false
log4j.logger.kafka.network.RequestChannel$=WARN, requestAppender
log4j.additivity.kafka.network.RequestChannel$=false

# controller 日志级别，默认为 TRACE
log4j.logger.kafka.controller=INFO, controllerAppender
log4j.additivity.kafka.controller=false

# 日志清理的日志级别
log4j.logger.kafka.log.LogCleaner=INFO, cleanerAppender
log4j.additivity.kafka.log.LogCleaner=false

log4j.logger.state.change.logger=TRACE, stateChangeAppender
log4j.additivity.state.change.logger=false

# 登陆认证的日志级别
log4j.logger.kafka.authorizer.logger=INFO, authorizerAppender
log4j.additivity.kafka.authorizer.logger=false
```

### 4. 配置 JVM 参数

_**Warning**：谨慎调试_

在 `bin/kafka-server-start.sh` 文件中调整参数如下：

```bash
# 在 base_dir 之后配置参数
base_dir=$(dirname $0)

export KAFKA_HEAP_OPTS="-Xms6g -Xmx6g -XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80"
```

_**ps：** 虽然看起来很激进，但是以上配置参照的是 LinkIn 高峰时期最繁忙的集群:_

```properties
60 brokers
50k partitions (replication factor 2)
800k messages/sec in
300 MB/sec inbound, 1 GB/sec+ outbound
```

_**ps2：** 这个配置的集群实现了 90% 的 GC 中断时间不超过 21 毫秒，每秒钟新生代 GC 次数不超过一次_

_**ps3：** 上述环境的 Java 版本为 JDK 1.8 u5_

### 5. 配置 Linux 参数

_**Warning**：谨慎调试_

```bash
# 调整系统所有进程一共可以打开的最大文件数：
echo 'fs.file-max = 1024000' >> /etc/sysctl.conf
```

以及 `/etc/security/limits.conf` 末尾增加：

```bash
# 设置当前 user 以及由它启动的进程的资源限制
{user}      soft    nofile      1024000
{user}      hard    nofile      1024000
```

增大 socket buffer size，以提高吞吐性能

```bash
echo 212992 >> /proc/sys/net/core/wmem_max
echo 212992 >> /proc/sys/net/core/rmem_max
```

## Kafka2.0 官方安装指南

[Quick Start](https://kafka.apache.org/documentation/#quickstart)
