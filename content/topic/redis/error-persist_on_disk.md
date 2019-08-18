---
title: "解决 not able to persist on disk"
linktitle: "not able to persist on disk"
toc: true
type: docs
date: "2018-05-18T11:42:50+08:00"
draft: false
menu:
  redis:
    parent: 问题处理
    weight: 2
    
---

## 报错信息

> MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk.

## 原因

绝大多数情况是写磁盘写满了，并且redis默认`stop-writes-on-bgsave-error`配置为`yes`，无法正确的存储rdb文件的时候也就拒绝客户端的请求了。

## 解决办法

### 解决rdb保存问题

有重要数据的时候不能直接清空重来，先检查磁盘空间是否足够，然后指定一个新的rdb文件，重新`bgsave`：

```bash
# 1. 更改工作目录位置
CONFIG SET dir /tmp/some/directory/other/than/var

# 2. 设置rdb文件名
CONFIG SET dbfilename temp.rdb

# 3. 保存新的rdb文件
BGSAVE
```

以上修改会在重启redis后失效，根据实际情况处理吧。

### (临时)解决无法写入问题

关闭rdb保存失败拒绝写入的功能：

```bash
config set stop-writes-on-bgsave-error no
```
