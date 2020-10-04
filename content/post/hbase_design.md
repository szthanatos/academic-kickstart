---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "hbase 表设计风格指南"
subtitle: ""
summary: ""
authors: []
tags: ['HBase']
categories: ['DataBase']
date: 2018-12-09T16:19:27+08:00
lastmod: 2018-12-09T16:19:27+08:00
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

{{% toc %}}

## 简介

本指南是对在 HBase 进行字段设计而提供的指导性准则和建议。总体标准、设计方式参照 [Google 开源项目风格指南](https://github.com/google/styleguide) 以及现有项目经验。所有条目均为个人总结，**并不是一份官方标准性质的指南** 。

HBase 是建立在 Hadoop 文件系统（HDFS）之上的分布式、面向列的数据库。

## 一般原则

- 无论是表或者是列或者其他，都应该使用名词或者动宾短语以代表一类对象;
- 尽量避免使用 (尤其是单独使用) 例如 `int`、`join`、`select` 等常见保留词;
- HBase 在性能和效率上更擅长处理 “高而瘦” 的表，而非 “矮而胖” 的表——以 Excel 类比，HBase 应该尽可能设计成只有很少的列 (瘦) 而有非常多行 (高) 的模式;

## 命名空间 (NameSpace)

### 命名规范

- 采用英文单词、阿拉伯数字的组合形式，其中，单词必须大写，并且首字符必须为英文字符，不能是数字;
- 不建议用连接符（下划线）拼接多个单词，简单语义的可采用单个单词，复杂语义的可采用多个单词的首字母拼接;
- 长度尽量限制在 4~8 字符之间;
- 命名空间一般可与项目名称、组织机构名称等保持一致;
- 一般情况下如果不指定命名空间，表会被放在默认 (default) 命名空间下;

### 示例

```bash
ZKR
XJ917
```

## 表 (Table)

### 命名规范

- 采用英文单词、阿拉伯数字、连接符（_）的组合形式，其中，单词必须大写，并且首字符必须为英文字符，可用连接符拼接多个单词;
- 长度尽量限制在 8~16 字符之间;
- 尽量采用具有明确意义的英文单词，而不建议采用汉字的拼音字母或者拼音首字母组合;
- 无需以 `TABLE` 结尾;

### 示例

```bash
USER_INFO
WEIBO_USER_FANS
```

## 行键 (Rowkey)

### 命名规范

- 采用英文单词、阿拉伯数字、非转义字符组合形式，不要求大小写，但首字符必须是英文字符或数字;

### 示例

```bash
123456-654321
dftf3a3l3rv3qr
s.taobo.com/faefavc
```

### 注意

#### 慎重将时间戳直接放入行键中

对于同一条数据，HBase 本身提供时间戳 (TimeStamp) 以在同一个 RowKey 下保存不同版本数据;
对于整体，存放旧数据的区域随着时间戳增大可能不再写入，而存放新数据的区域始终保持高负荷，这样降低了 HBase 整体的读写能力。

一个推荐的方式是使用反向时间戳。

#### 权衡 hash 和 string 的效果

哈希化 (一般特指单项哈希) 的 Rowkey 能很好的避免热点问题，但是也会同时丢失直接使用 String 的 RowKey 的天然聚类和排序的能力。

## 列族 (ColumnFamily)

### 命名规范

- 采用英文单词、阿拉伯数字的组合形式，其中，单词必须大写，并且首字符必须为英文字符，不能是数字;
- 长度尽量限制在 1~6 字符之间，过长的列族名称将占用更多的存储空间, 它们不应该像在典型的 RDBMS 中一样具有自我记录和描述性;

### 示例

```bash
DATA
D1 # data1
WA # web args
```

### 注意

#### 列族的数量应控制在 1-3 个

HBase 表不应该被设计成模拟 RDBMS 表，列族的数量在满足需求的情况下应该尽可能少。在存储时，一个列族会存储成一个 StoreFile，多个列族对应的多个文件在分裂时会对服务器造成更大的压力。

## 列 (Qualifier)

### 命名规范

- 采用英文单词、阿拉伯数字、连接符（_）的组合形式，其中，单词必须 ** 小写 **，并且首字符必须为英文字符，不能是数字，可用连接符拼接多个单词;
- 所有列名都应该是名词或者以 `is` 开头的动宾短语 (表示判断)，不应该使用其他词性单词;
- 允许使用前缀，不允许使用后缀;
- 长度尽量限制在 1~16 字符之间;
- 尽量采用具有明确意义的英文单词，而不建议采用汉字的拼音字母或者拼音首字母组合;

### 示例

```bash
user_name
is_str
sound_type
```
