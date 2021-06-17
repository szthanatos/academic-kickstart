---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "指北系列指南"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-12-28T19:03:14+08:00
lastmod: 2020-12-28T19:03:14+08:00
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

我觉得网上技术教程都很有坑。

比如官方文档。如果只想尽快使用上某种技术产品，入门章节一般都是实验级别的，不太够用，而其余章节就有点过于细节;

要么是零碎的个人笔记，基本只写自己关心的那一部分，叙述角度和内容都非常零散。

所以我到底想要什么样的教程呢？

对于我个人而言，我期望的是一个比个人笔记长，比官方文档短的一个教程，读起来不会太累；

内容可以不深入，但是要系统和全面，看完了能让读者有一个整体的概念；

而对于不同领域的知识，通过同样的叙述方式，降低认知负担，快速定位自己关心的部分；

教程都统一叫指北，不全面也不权威，和系统学习官方文档背道而驰，但是30分钟建立一个感性认识完全足够了。

所以我拟定了这样的一个思路：

![指北系列目录结构](https://i.loli.net/2021/06/17/hZ1TLXpDWY3v9UA.png)

作为我个人博客的指北专题的固定格式，

也期望通过这样的方式更好的帮助自己建立和分享知识体系。

生成目录的脚本如下：

```bash
#!/bin/bash
create_page(){
    cat > ${1}/${2}.md <<EOF
---
title: 
linktitle: ${3}
summary: 
date: $(date '+%Y-%m-%d %H:%M:%S')
lastmod: $(date '+%Y-%m-%d %H:%M:%S')
draft: false
toc: true
type: book
weight: ${4}
---
EOF
}

doc_title="$1"
mkdir -p content/$1
cd content/$1

dir_list=( 
    01-design 
    02-useage 
    03-operation 
    04-advanced 
    05-troubleshooting 
    )

for chapter in ${dir_list[*]}
do
    mkdir -p $chapter
done

# chapter
create_page "${PWD}" _index "简介" 1
create_page 01-design _index "设计" 10
create_page 02-useage _index "使用" 40
create_page 03-operation _index "运维" 70
create_page 04-advanced _index "进阶" 100
create_page 05-troubleshooting _index "排错" 130

# page
create_page 01-design concept "概念" 10
create_page 01-design architecture "架构" 20
create_page 01-design component "组件" 30
create_page 02-useage install-and-config "安装/配置" 40
create_page 02-useage operate "操作" 50
create_page 02-useage example "用例" 60
create_page 03-operation update-and-rollback "升级/降级" 70
create_page 03-operation backup-and-restore "备份/恢复" 80
create_page 03-operation observability "观测性" 90
create_page 04-advanced optimize "优化" 100
create_page 04-advanced trick"技巧" 110
create_page 04-advanced reference "参考" 120

cd ../../

```

如果你也使用 `hugo` 和 `wowchemy`，那么在项目主目录执行 `./create_book.sh [文档名]` 就会在 `content/[文档名]` 下生成所有相关文件和文件夹，并配置好目录顺序。
