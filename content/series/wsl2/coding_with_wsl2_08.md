---
title: Pycharm 易用性设置
linktitle: 配置 Pycharm
toc: true
type: book
date: 2020-08-22T21:51:08+08:00
lastmod: 2020-08-22T21:51:08+08:00
draft: false
weight: 80
---

## 选项

### 大小写敏感

`File` -> `Settings` -> `Editor` -> `General` -> `Code Completion`

取消后全小写也能自动补全，不过为了养成好习惯建议还是开首字母。

### 行分隔符

`File` -> `Settings` -> `Editor` -> `Code Style`

Windows 下默认换行是 `CRLF` 即 `\r\n`，改成 Unix 的 `LF`(`\n`)

### 头部模板

`File` -> `Settings` -> `Editor` -> `File and Code Templates` -> `Python Script`

写入以下内容作为 Python 默认头部模板：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
@Author : ${USER}
@File   : ${NAME}.py
@Project: ${PROJECT_NAME}
@Time   : ${DATE} ${TIME}
"""

```

`${USER}` 是系统用户名，你也可以写死。

### 鼠标缩放

`File` -> `Settings` -> `Editor` -> `General` -> `Change font size(Zoom) with Ctrl+Mouse Wheel`

按住 `Ctrl` 滚轮缩放。

## 插件

### 市场插件

- `.env file support`: .env 文件格式支持
- `.ignore`: git .ignore 模板
- `Chinese (Simplified) Language Pack EAP`: 汉化
- `CodeGlance`: Sublime 式的小地图
- `Key Promoter X`: 使用功能时显示对应快捷键
- `Rainbow Brackets`: 不同层级括号不同颜色
- `Requirement`: 自动检查 Requirement 文件是否和实际环境匹配
- `Translation`: 翻译插件

### 外部插件

另外建议使用 [Black](https://github.com/psf/black) 让强制统一代码风格，多人协作的时候非常有用。

运行 Black 需要 Python 环境，到底放 Windows 还是 Wsl 还是 Docker 都行，看你有多骚了。

集成步骤直接按 [文档](https://black.readthedocs.io/en/stable/editor_integration.html) 走就好。摘抄如下：

1. `pip install black`
2. 记录 Black 位置

    Windows 下可能在 `C:\Users\{用户名}\AppData\Local\Programs\Python\{Python 版本}\Scripts\black.exe`

   Linux 和 MacOS 可能在 `/usr/local/bin/black`

3. `File` -> `Settings` -> `Tools` -> `External Tools`，新建内容如下：

    ```txt
    Name: Black
    Description: Black is the uncompromising Python code formatter.
    Program: <第二步的位置>
    Arguments: "$FilePath$"
    ```

    建议取消 `Advanced Options` 中的 ` 打开工具输出控制台 `

4. 配置快捷键，我建议是直接替换 `Ctrl+Alt+l` 格式化
5. 监视文件更改自动执行，在 `File` -> `Settings` -> `Tools` -> `File Watchers` 下新建：

    ```txt
    Name: Black
    File type: Python
    Scope: Project Files
    Program: <第二步的位置>
    Arguments: $FilePath$
    Output paths to refresh: $FilePath$
    Working directory: $ProjectFileDir$
    ```

    建议不勾选 `Advanced Options` 中的 `Auto-save edited files to trigger the watcher`

## 快捷键

`File` -> `Settings` -> `Keymap` -> `Editor Action`

我用 Pycharm 默认键位很习惯，只加了:

- 向左: `Ctrl + ;`
- 向右: `Ctrl +  Shift + ;`
