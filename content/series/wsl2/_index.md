---
title: WSL2 开发环境 30 分钟指北
linktitle: 介绍
summary: Wsl2+Ubuntu+Zsh+Tmux+Docker
# date: "2018-09-09T00:00:00Z"
lastmod: "2020-09-09T00:00:00Z"
draft: false  # Is this a draft? true/false
toc: true  # Show table of contents? true/false
type: book  # Do not modify.
weight: 1
---

> 2020 年，我使用的 Windows 系统被评为最好的 Linux 发行版 ←_←
>
> 为什么呢？
>
> 各类工具层出不穷
>
> 环境生态市场份额世界第一
>
> 这是事实，无需否认
>
> 所以人们蜂拥而来
>
> 这系统总会给你一丝希望
>
> 谎言也好，幻觉也罢
>
> 但如此近
>
> 仿佛触手可及
>
> 让人奋不顾身...

在这个程序员标配苹果的世界里，对于那些对 MacOS 抱有偏见的人来说（比如我），Wsl2 就是那丝希望。

不过讲道理，相对于其他平台（主要是 MacOS），想要在 Windows 下获得足够舒适（颜值在线）的开发体验，过程还是繁琐许多，无法开箱即用，可能需要一些技巧才能上手。

公平的是，配置好之后，得益于 Windows 生态，体验的上限也比其他平台要高（个人意见）。

我这边主要是后端开发（Python most, Go sometimes）为主，代码也主要运行在 Linux 环境下。

想要获得完整的开发体验，你需要：

{{<list_children>}}

请根据实际需要进行针对性配置。

比如不整这些花里胡哨的，直接配置 Wsl 和 Docker 就好...

{{< spoiler text="又不是不能用... ┑(￣Д ￣)┍" >}} 异端~ 颜值才是生产力！<p>(╯‵□′)╯︵┻━┻ {{< /spoiler >}}
