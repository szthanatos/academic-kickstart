---
title: Kafka -02- 滚动升级
linktitle: 滚动升级
toc: true
type: book
date: 2018-12-09T15:19:52+08:00
draft: false
weight: 20
---

## 环境说明

|              | 版本号   | 发布日期   |
| ------------ | -------- | ---------- |
| **当前版本** | 0.11.0.1 | 2017-09-14 |
| **最新版本** | 2.0      | 2018-07-30 |

**配置文件路径：**
> /home/tools/kafka_2.12-0.11.0.1/config/

**目标需求：** 在 Kafka 集群不停机不停止服务的情况下进行升级改造。

## 可能存在的风险

### 轻微警报

- consumer 可能出现偏移量提交失败而造成重复消费
- broker 提示'NotLeaderForPartitionException'异常
- 由于节点下线，可能造成临时性能问题

### 严重警报

严格按照步骤升级，暂未捕捉到严重问题相关信息

## 升级步骤（滚动升级）

- 限定通讯协议版本：

    配置 broker 上的 server.properties 文件：

    ```ini
    inter.broker.protocol.version = 0.11.0
    ```

- 依次更新代码并重启 borker：

    一次关闭一个 broker，更新源码，重启

- 更新通讯协议版本：

    完成所有 broker 节点的源码更新后, 升级协议（方法同上）：

    ```ini
    inter.broker.protocol.version = 2.0
    ```

- 再次依次重启 broker：

    同上，一次重启一个

**_ps：_** 如果修改过消息格式版本 (log.message.format.version)，则需要在上面步骤中，同步配置：

```ini
log.message.format.version = 当前版本 / 要升级的版本
```

## 替代方案（离线升级）

关闭所有 broker，更新代码并重新启动。默认情况下，自动以新协议开始。

## Kafka2.0 官方升级指南

[upgrade](https://kafka.apache.org/documentation/#upgrade)
