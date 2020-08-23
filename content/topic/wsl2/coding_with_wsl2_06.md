---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: 直接使用Ubuntu作为Python开发环境
linktitle: Python环境配置
toc: true
type: docs
date: 2020-08-22T18:06:35+08:00
lastmod: 2020-08-22T18:06:35+08:00
draft: false
menu:
  wsl2:
    weight: 7

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 7
---

在前面的部分已经配置好了Python的死蛇源，所以现在已经可以使用Ubuntu作为Python基础环境了。

## 基本配置

安装最新版本Python：

```bash
sudo apt install python3.x
sudo apt install python3.x-distutils
sudo apt install python3-pip
```

修改pypi源，使用清华大学的镜像：

```bash
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## Pycharm连接

Pycharm 2019.1.X+ 版本已经支持直接连接wsl的python环境，无需通过ssh

![pycharm添加wsl解释器](https://raw.githubusercontent.com/szthanatos/image-host/master/pycharm-wsl.png)
