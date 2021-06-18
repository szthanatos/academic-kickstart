---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "python 单元测试标准及实现"
subtitle: ""
summary: ""
authors: []
tags: ['unit testing']
categories: ['Python']
date: 2018-08-20T10:49:50+08:00
lastmod: 2018-08-20T10:49:50+08:00
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

{{% callout note %}}
这是写给我组里的人看的，顺手粘过来
{{% /callout %}}

## 什么是单元测试

> 单元测试 (Unit Testing) 又称为模块测试，是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。程序单元是应用的最小可测试部件。在过程化编程中，一个单元就是单个程序、函数、过程等；对于面向对象编程，最小单元就是方法，包括基类（超类）、抽象类、或者派生类（子类）中的方法。

一句话概括，单元测试也就是校验代码中具体的类 (甚至函数) 的输出值是否符合预期。

## 为什么要写单元测试

> “可能出错的事情最终一定会出错”
>
> ——墨菲定律

代码随着时间的累计而增长，出现意想不到的问题的可能性也在指数级上升。代码的正确与否不应该靠人来保证，因为人是会犯错并且一定犯错的。如果每次新功能上线时不能回答 “所有功能都测试过了么” 的问题，那么最终整个项目的可靠性都将被摧毁。单元测试的意义就在于让你能够回答这个问题，并且，回答的更自动化。

## 怎么写单元测试

### 原生测试框架 unittest/unittest2

在 python 语境中，官方提供 unittest 标准库完成单元测试。

#### 基础

需要理解的概念有如下四个：

- `test fixture`：单元测试所需上下文环境，比如临时数据库 / 网络连接等；
- `test case`：一个独立的单元测试最小单位；
- `test suite`：test case 的集合；
- `test runner`：执行并输出单元测试的程序；

详细定义请自行查阅 [官网文档](https://docs.python.org/3/library/unittest.html)。

官方单元测试用例如下，我们对 upper(将 string 转换为大写)、isupper(判断 string 是否全部为大写)、split(对 string 按空格切分为 list) 函数的功能进行校验

```python
import unittest

class TestStringMethods(unittest.TestCase):

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())

    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        # check that s.split fails when the separator is not a string
        with self.assertRaises(TypeError):
            s.split(2)

if __name__ == '__main__':
    unittest.main()
```

#### 断言

可以从上面的例子看出，单元测试判断结果是否符合预期的主要方式是通过断言 (assert)  实现。
以下是常见断言：

| Method                     | Checks that          |
| -------------------------- | -------------------- |
| assertEqual(a, b)          | a == b               |
| assertNotEqual(a, b)       | a != b               |
| assertGreater(a, b)        | a > b                |
| assertGreaterEqual(a, b)   | a >= b               |
| assertLess(a, b)           | a < b                |
| assertLessEqual(a, b)      | a <= b               |
| assertAlmostEqual(a, b)    | round(a-b, 7) == 0   |
| assertNotAlmostEqual(a, b) | round(a-b, 7) != 0   |
| assertRegex(s, r)          | r.search(s)          |
| assertNotRegex(s, r)       | not r.search(s)      |
| assertTrue(x)              | bool(x) is True      |
| assertFalse(x)             | bool(x) is False     |
| assertIs(a, b)             | a is b               |
| assertIsNot(a, b)          | a is not b           |
| assertIsNone(x)            | x is None            |
| assertIsNotNone(x)         | x is not None        |
| assertIn(a, b)             | a in b               |
| assertNotIn(a, b)          | a not in b           |
| assertIsInstance(a, b)     | isinstance(a, b)     |
| assertNotIsInstance(a, b)  | not isinstance(a, b) |

#### setUp 和 tearDown

`test fixture` 是通过 `setUp` 和 `tearDown` 来具体实现的。

**setUp() 方法**：
在执行每个测试用例 (test case) 之前被执行，除了 unittest.SkipTest 和 AssertionError 以外的任何异常都会当做是 error 并终止当前测试用例；

**tearDown() 方法**：
执行了 setUp()方法后，执行 tearDown()方法 (进行清理)。对异常的处理和 setUp() 类似；

**setUpClass(cls) 与 tearDownClass(cls) 类**：
可以将 setUp 和 tearDown 定义在基类中避免重复定义，定义 setUpClass(cls) 与 tearDownClass(cls) 类时必须加上 classmethod 装饰符；

对上面的例子进行简单的改造以演示 setUp 和 tearDown 的效果：

```python
import unittest

class TestStringMethods(unittest.TestCase):

    def setUp(self):
        print '1. setUp here'

    def tearDown(self):
        print '2. tearDown here'

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    ...
```

执行后效果如下：

```shell
test_isupper (mytest.TestStringMethods) ... 1. setUp here
1. tearDown here
ok
```

#### 缺点

基本的一个单元测试可以用这四步概括：

1. 新建单元测试脚本
2. 导入单元测试依赖
3. 继承单元测试类
4. 实现单元测试方法

而这个过程非常不 pythonic：

- 必须新建单独的测试文件
- 测试必须继承自 unittest 类，即使再简单的测试
- 断言只能使用 unittest 的 Assertion
- 最最关键和难以忍受的：unitunit 内的命名规则和 pep 8 相悖

造成这些问题的原因一言以蔽之：python 的测试框架是完全仿照 Java 实现的。

### 第三方测试框架 py.test

实际上，通过使用 py.test，我们可以非常 pythonic 的实现单元测试：

```python
# content of test_sample.py
def inc(x):
    return x + 1


def test_answer():
    assert inc(3) == 5
```

直接在测试文件所在目录执行 py.test 得到如下结果：

```shell
$ pytest
=========================== test session starts ============================
platform linux -- Python 3.x.y, pytest-3.x.y, py-1.x.y, pluggy-0.x.y
rootdir: $REGENDOC_TMPDIR, inifile:
collected 1 item

test_sample.py F                                                     [100%]

================================= FAILURES =================================
_______________________________ test_answer ________________________________

    def test_answer():
>       assert inc(3) == 5
E       assert 4 == 5
E        +  where 4 = inc(3)

test_sample.py:6: AssertionError
========================= 1 failed in 0.12 seconds =========================
```

就是这么简单，更进一步的，py.test 支持自动生成对指定目录下所有测试文件的统一测试脚本，更具体的用法参见 [pytest 文档](https://docs.pytest.org/en/latest/)

总的来说，py.test 具有如下特点：

- 非常容易上手，入门简单，文档丰富，文档中有很多实例可以参考
- 能够支持简单的单元测试和复杂的功能测试
- 支持参数化
- 执行测试过程中可以将某些测试跳过，或者对某些预期失败的 case 标记成失败
- 支持重复执行失败的 case
- 支持运行由 nose , unittest 编写的测试 case
- 具有很多第三方插件，并且可以自定义扩展
- 方便的和持续集成工具集成

## 单元测试标准

业界通常使用代码覆盖 (率) 来评判测试的好坏。

### 代码覆盖指标

单独的一两个测试完全无法体现测试的优势。而对所有可能的情况编写单元测试既不现实也无必要。所以明确测试覆盖哪些指标非常重要。**我们在此指定以下四个指标必须被覆盖：**

- **函数覆盖（Function Coverage）**

    每一个函数都必须被测试；

- **语句覆盖（Statement Coverage）**

    被测代码中每个可执行语句都应该被执行测试。例如

    ```python
    def foo(x:int, y:int):
        z = 0
        if x>0 and y >0:
            z = x
        return z
    ```

    中，如果测试为 assertEqualst(0, foo(2,-1))，则 if 内的代码就没有被覆盖到；

- **决策覆盖（Decision Coverage）**

    指每一个逻辑分支都应该被测试覆盖。类似上面的例子，如果想要达到决策覆盖，我们起码应该执行两次测试：

  - assertEquals(2, foo(2, 2))  # 决策 1
  - assertEqualst(0, foo(2,-1))  # 决策 2

- **条件覆盖（Condition Coverage）**

    每一个逻辑分支的每一个条件都应该被覆盖。条件覆盖不需要满足条件表达式所有的排列组合，而只需将每个条件表达式的结果为 true/false 的情况进行测试就可以了。依旧使用上面的例子，如果想要达到条件覆盖，我们应该执行至少三次测试：

  - assertEquals(2, foo(2, 2))  # 决策 1 条件 true
  - assertEqualst(0, foo(2,-1))  # 决策 2(没有条件)
  - assertEquals(0, foo(-1, -1))  # 决策 1 条件 false

    如果没有第三个测试，那么只能达到决策覆盖，不能达到条件覆盖。

### 代码覆盖率

在满足代码覆盖指标的基础上，只有保证一定的代码覆盖率才能保证测试的完整。满足代码覆盖指标相当于是 “质”，而代码覆盖率则是保证 “量”。**目前要求代码覆盖率不应该低于 75%**

我们选定 [coverage.py](https://pypi.org/project/coverage/) 来统计代码覆盖率。由于主要使用 py.test，需要额外安装 [pytest-cov](https://github.com/pytest-dev/pytest-cov) 插件。安装过程非常简单，对照文档直接 pip 安装即可，不多介绍。

完成安装后，使用 py.test 的时候增加 --cov=myproj 参数即可。
效果如下：

```shell
-------------------- coverage: ... ---------------------
Name                 Stmts   Miss  Cover
----------------------------------------
myproj/__init__          2      0   100%
myproj/myproj          257     13    94%
myproj/feature4286      94      7    92%
----------------------------------------
TOTAL                  353     20    94%
```

详细用法可参照 [官方文档](http://pytest-cov.readthedocs.io/en/latest/)

## 小结

总结一下，通过对单元测试的必要性、编写方法、评判标准等一系列的介绍，确立了以下三点：

- 使用 py.test+unittest 编写单元测试，使用 coverage 统计、分析单元测试编写情况；
- 单元测试应覆盖最基本的四项指标 (函数覆盖、语句覆盖、分支覆盖、条件覆盖)；
- 在覆盖基本指标的基础上，需要达到 75% 的代码覆盖率；
