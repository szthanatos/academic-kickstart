---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "connection reset by peer"
# linktitle: "connection reset by peer"
toc: true
type: book
date: 2019-06-16T23:06:41+08:00
draft: false
---

## 报错信息

> Error: Connection reset by peer

## 原因

读写操作发生在连接断开后。

## 解决办法

1. 用 `info`/`cluster info` 命令检查连接数是否过多
2. 检查 redis-server 是否正确监听配置文件
3. 检查配置文件中 `bind` 部分，如果是 `bind 127.0.0.1` 则只允许本地访问
4. 检查配置文件中 `protected-mode` 部分是否为 `no`
5. 重启 redis-server
