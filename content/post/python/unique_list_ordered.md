---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "列表原序去重性能测试"
subtitle: ""
summary: ""
authors: []
tags: ['list', 'benchmark']
categories: ['Python']
date: 2017-09-01T18:08:57+08:00
lastmod: 2017-09-01T18:08:57+08:00
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

对列表的去重很简单，`set()` 一下再 `list()` 回来就可以了，但是如果要保留原始列表的顺序呢？

举例，对 `["b", "b", "c", "a", "c", "b", "a", "b"]` 这个列表进行原序去重，得到结果应该是 `['b', 'c', 'a']`。

有下面这几种写法：

#### 二次排序

也就是对去重结果再按原列表 sort 一次：

```python
def sort_1(list_in):
    return sorted(list(set(list_in)), key=list_in.index)
```

#### 匿名函数

使用匿名函数将列表里不重复的元素累加到一个新列表中：

```python
def sort_2(list_in):
    return reduce(lambda x, y: x if y in x else x + [y], [[], ] + list_in)
```

#### 借用字典

##### 有序字典

使用 `OrderedDict` 排序：

```python
def sort_3(list_in):
    return list(collections.OrderedDict.fromkeys(list_in).keys())
```

#### defaultdict

类似的, 我们使用 `defaultdict` 进行排序：

```python
def sort_4(list_in):
    return list(collections.defaultdict.fromkeys(list_in).keys())
```

#### 直接使用 dict

在 python3.6 之前， `dict` 的 `key` 的顺序并不保证一定是插入顺序，所以只有在 python3.6 之后才可以直接用 `dict` 实现这个操作；

```python
def sort_5(list_in):
    return list(dict.fromkeys(list_in).keys())
```

完整性能测试代码如下：

```python
# !/usr/bin/env python
# encoding: utf-8

from timeit import repeat
from functools import reduce
from collections import defaultdict, OrderedDict

example = ["b", "b", "c", "a", "c", "b", "a", "b"]


def sort_1(list_in):
    return sorted(list(set(list_in)), key=list_in.index)


def sort_2(list_in):
    return reduce(lambda x, y: x if y in x else x + [y], [[], ] + list_in)


def sort_3(list_in):
    return list(OrderedDict.fromkeys(list_in).keys())


def sort_4(list_in):
    return list(defaultdict.fromkeys(list_in).keys())


def sort_5(list_in):
    return list(dict.fromkeys(list_in).keys())


if __name__ == '__main__':
    # time usage: t5< t4 < t3 < t2 < t1
    result = {}
    for i in range(1, 6):
        result['sort_{}'.format(i)] = repeat('sort_{}(example)'.format(i),
                                             'from __main__ import sort_{}, example'.format(i),
                                             number=1000000,
                                             repeat=5)
    for k, v in result.items():
        avg_v = round(sum(v) / len(v), 3)
        print(k, avg_v)

```

在我的苏菲婆上的结果仅供参考：

| 排序   | 平均时间 |
| ------ | -------- |
| sort_1 | 1.477    |
| sort_2 | 1.305    |
| sort_3 | 0.957    |
| sort_4 | 0.734    |
| sort_5 | 0.698    |

可见，python3.6 之后 dict 是最好的原序去重办法，3.6 之前用 defaultdict 吧。
