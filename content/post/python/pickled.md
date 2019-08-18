---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "可序列化类型和多进程PicklingError"
subtitle: ""
summary: ""
authors: []
tags: ['PicklingError', '序列化', '多进程']
categories: ['Python']
date: 2018-03-20T16:16:15+08:00
lastmod: 2018-03-20T16:16:15+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

遇到一个报错：

> `PicklingError: Can't pickle <type 'instancemethod'>: attribute lookup __builtin__.instancemethod failed`

当时的情况是想写一个多进程的解析代码，爬虫爬到的内容给扔过来就不管了，差不多这个意思：

```python
# !/usr/bin/env python
# -*- coding: utf-8 -*-

from concurrent.futures import ProcessPoolExecutor


class PageProcess(object):
    def __init__(self, worker):
        self.max_worker = worker

    def single_process(self, page):
        pass

    def multi_process(self, page_list):
        with ProcessPoolExecutor(max_workers=self.max_worker) as pp:
            result = pp.map(self.single_process, page_list)

```

这个错误是这么造成的：

1. 在类中使用进程池；
2. 进程池使用`Queue`管理任务队列；
3. `Queue`要求传递的内容必须都是可以被序列化的；

那么问题来了，哪些类型是可以被序列化的呢？

根据[官方文档](https://docs.python.org/3/library/pickle.html#pickle-picklable)，可序列化的类型包括：

| 类型                                                       | 原文                                                                                              |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 布尔型和空值                                               | `None`, `True`, and `False`                                                                       |
| 数字类型中的整数，浮点数和复数                             | integers, floating point numbers, complex numbers                                                 |
| 字符串类型和二进制类型(字节流，字节数组)                   | strings, bytes, bytearrays                                                                        |
| 只包含可序列化对象的元组、集合、列表、字典                 | tuples, lists, sets, and dictionaries containing only picklable objects[^1]                       |
| 模块中最顶层声明的非匿名函数                               | functions defined at the top level of a module (using def, not lambda)                            |
| 模块中最顶层声明的内置函数                                 | built-in functions defined at the top level of a module                                           |
| 模块中最顶层声明的类                                       | classes that are defined at the top level of a module                                             |
| `__getstate__`的结果或`__dict__`是可序列化的这样的类的实例 | instances of such classes whose `__dict__` or the result of calling `__getstate__()` is picklable |

破案了，上面代码中，我们的进程池要序列化的是类中的函数，就不符合最顶层定义的函数的要求。

所以最直接的解决办法也很简单，把要并行的函数抽外面去就行了：

```python
# !/usr/bin/env python
# -*- coding: utf-8 -*-

from concurrent.futures import ProcessPoolExecutor


def single_process(page):
    pass


class PageProcess(object):
    def __init__(self, worker):
        self.max_worker = worker

    def multi_process(self, page_list):
        with ProcessPoolExecutor(max_workers=self.max_worker) as pp:
            result = pp.map(single_process, page_list)

```

[^1]: 英文这种语序/标点我老是搞不懂，这个`containing only picklable objects`到底是指dictionaries还是前面全部，就当是全部吧
