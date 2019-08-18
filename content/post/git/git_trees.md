---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Git三棵树和reset/checkout命令"
subtitle: ""
summary: ""
authors: []
tags: ['Git']
categories: []
date: 2019-01-05T14:13:21+08:00
lastmod: 2019-01-05T14:13:21+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false
  # [header]
  #   image = "batman-cave.jpg"
  #   caption = ""

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

理解Git没有比从三棵树开始更好的了。

完整的话还是看[git文档](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)比较好，说的蛮清楚了。

## 三棵树和正向流程

| 树                | 用途                                 |
| ----------------- | ------------------------------------ |
| HEAD              | 上一次提交的快照，下一次提交的父结点 |
| Index             | 预期的下一次提交的快照               |
| Working Directory | 沙盒                                 |

git的核心工作就是管理这三棵树。`git add`就是把你工作目录(Working Directory)的修改提交到暂存区(Index)，`git commit`就是把暂存区的内容同步到仓库里作为一个快照，并移动`HEAD`指向新快照；![git_flow](/img/git_flow.gif)

额外说一下这个`HEAD指针`，每一次commit都相当于在仓库(Repository)里生成一个快照，
把N个快照想象成一个右进左出的队列(List)，再想象有一个指针，默认指向队首(最新快照)，告诉你当前到底用的是哪一个版本快照。

## reset

显然，`git reset`就是对上述行为的反向操作。

`reset`的本质其实是移动`HEAD`指针指向哪个快照，而通过

- `--soft`——只改变指针指向的快照；
- `--mixed`——移动指针的同时也把快照内容同步到暂存区；
- `--hard`——三棵树全同步为指针指向的快照；

参数来**递进的**控制改变是发生在哪几颗树上。![git_reflow](/img/git_reflow.gif)

再强调一遍，`reset`的改变的是`HEAD`指针，而不是文件。即使`git reset File`的写法是有效的，但它的本质是`git reset --mixed HEAD File`的缩写，即将`File`从`HEAD`指向的快照复制到索引中。`HEAD`指针永远只能指向一个快照，但是快照是可以局部修改它里面的文件的。

`HEAD~`表示前一个快照，`HEAD~2`表示前两个，依此类推。

## checkout

`checkout`的本质就有所不同，它关心的是分支(branch)，它的主要作用是让`HEAD`在不同分支间移动(默认三棵树都会更新)。

还是拿刚才那个队列举例，分支相当于是平行的一条队列，现在把他放在你脑子里之前那个队列的上方，
由于`HEAD`指针只能指向一个快照，所以这个时候它可能会在两个队列间“跳动”，`checkout`就是控制指针上下移动的命令，而`reset`则是控制指针在当前队列左右(前后)移动。![git_chechout](/img/git_checkout.gif)

同样的，`checkout`后面也可以跟一个文件，和`git reset --hard [branch] file`可能会产生的效果一样。
