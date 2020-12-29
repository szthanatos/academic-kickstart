---
title: 直接使用 Ubuntu Python 作为开发环境
linktitle: 配置 Python 环境 (选) 
toc: true
type: book
date: 2020-08-22T18:06:35+08:00
lastmod: 2020-08-22T18:06:35+08:00
draft: false
weight: 60
---

在前面的部分已经配置好了 Python 的死蛇源，所以现在已经可以使用 Ubuntu 作为 Python 基础环境了。

## 基本配置

安装最新版本 Python：

```bash
sudo apt install python3.x
sudo apt install python3.x-distutils
sudo apt install python3-pip
```

修改 pypi 源，使用清华大学的镜像：

```bash
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## Pycharm 连接

Pycharm 2019.1.X+ 版本已经支持直接连接 wsl 的 python 环境，无需通过 ssh

![pycharm 添加 wsl 解释器](https://raw.githubusercontent.com/szthanatos/image-host/master/pycharm-wsl.png)
