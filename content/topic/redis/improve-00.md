---
title: "优化指南"
linktitle: "优化指南"
toc: true
type: docs
date: 2019-03-18T15:04:35+08:00
draft: false
menu:
  redis:
    weight: 3

weight: 3
---

redis性能优化可以从设计，使用，运维三个层面上着手：

- **设计**
  - **结构**：合理选择数据类型
  - **负载**：保证数据在空间上均匀分布
- **使用**，
  - **操作**：保证操作在时间上均匀分布
  - **策略**：优化资源利用
- **运维**
  - **监控**：持续关注执行效率和性能指标
  - **部署**：物理层面的调优

下面具体的谈一下几个tips。