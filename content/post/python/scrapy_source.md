---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Scrapy 源码解读"
subtitle: ""
summary: ""
authors: []
tags: ['scrapy', 'spider']
categories: ['Python']
date: 2019-02-15T14:35:09+08:00
lastmod: 2019-02-15T14:35:09+08:00
featured: false
draft: true

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

在 Scrapy 中，数据是这样流动的：

## 数据流程

![Data flow](https://i.loli.net/2021/06/17/uQSltfjKVO3Xo6g.png)

1. ` 引擎 ` 从 ` 爬虫 ` 中获取待采集的初始化 **请求**(将待访问的 url 用 Requests 对象包裹起来)；
2. ` 引擎 ` 把 **请求** 放入 ` 调度器 ` 中，并向 ` 调度器 ` 申请下一个待采集的 **请求**(这不是多此一举，两次的 **请求** 可能不一样，并且执行起来也是异步不相干的)；
3. ` 调度器 ` 将待采集的 **请求** 返回给 ` 引擎 `；
4. ` 引擎 ` 通过一组 ` 下载器中间件 `(Scrapy 自带的有 14 个)，将 **请求** 发送给 ` 下载器 `；
5. 当这个 **请求** 对应的页面被下载完成之后，` 下载器 ` 会生成一个 **响应**(Response)，并再次通过 ` 下载器中间件 ` 将之返回给 ` 引擎 `；
6. ` 引擎 ` 从 ` 下载器 ` 那接收到返回的 **响应**，并将它通过一组 ` 爬虫中间件 `(Scrapy 自带 5 个) 发送给 ` 爬虫 ` 进行处理；
7. ` 爬虫 ` 处理完这个 **响应**，将解析出的 **项目容器**(Items) 以及新的 **请求** 通过 ` 爬虫中间件 ` 发送给 ` 引擎 `；
8. ` 引擎 ` 把 ` 爬虫 ` 返回的 **项目容器** 交由 ` 项目管道 `(Item Pipelines) 进行处理 (持久化)，之后将新的 **请求** 发送给 ` 调度器 `，并申请下一个要采集的 **请求**(如果存在的话)；
9. 1-8 的处理过程会不断重复，直到 ` 调度器 ` 中没有新的 **请求** 为止；

## 组件 & 源码

下面就上文涉及到的组件分别对作用和源码进行简单解读：

| 组件                   | 含义         | 作用                                                                                                                       |
| ---------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------- |
| Scrapy Engine          | 引擎         | 负责控制系统所有组件之间的数据流，并在发生某些操作时触发事件。                                                             |
| Scheduler              | 调度器       | 接收来自引擎的请求，并将之排入队列，以方便之后在引擎请求时调度。                                                           |
| Downloader             | 下载器       | 负责获取网页并将其提供给引擎，引擎又将它们提供给爬虫。                                                                     |
| Spiders                | 爬虫         | 由用户编写的自定义类，用于解析响应并从中提取项目或产生新的请求。                                                           |
| Item Pipeline          | 项目管道     | 负责处理爬虫产生的 Item，典型的任务包括清理，验证和持久化（如将 Item 存储在数据库中）。                                    |
| Downloader middlewares | 下载器中间件 | 位于 Engine 和 Downloader 之间的特定钩子，处理从 Engine 传递到 Downloader 的请求，以及从 Downloader 传递回 Engine 的响应。 |
| Spider middlewares     | 爬虫中间件   | 位于 Engine 和 Spider 之间的特定钩子，能够处理爬虫的输入（响应）和输出（项目和请求）。                                     |
