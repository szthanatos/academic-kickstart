---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Git Tips"
subtitle: ""
summary: ""
authors: []
tags: ['Git']
categories: []
date: 2018-12-14T09:35:32+08:00
lastmod: 2018-12-14T09:35:32+08:00
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

## Deploy key/SSH key(github)

`Deploy key` 是在 ` 项目主页 `-`setting`-`Delpoy keys` 下进行添加，如果勾选 `Allow write access`，则相当于具有对这个项目的读写权限 (否则只能 `clone` 不能 `push`)。**作用范围是这个项目。**

`SSH key` 是在你 ` 个人主页 `-`Settings`-`SSH and GPG keys` 下进行添加。**作用范围是你的账户下的所有项目。**

同一个 ` 公钥 `，只能作为整个账户的 `SSH key`，或者 **一个项目** 的 `Deploy key`。想为一台机器授予多个项目的读写权限的话，需要通过 `ssh-keygen` 生成多个密钥，分别作为不同项目的 `Deploy key`。

## 更换 git 协议

使用 `http`/`https` 协议连接仓库相比 `ssh` 即不够安全，也会存在 `push` 的时候必须输入用户名密码的问题。

使用 `git remote -v` 可以查看项目使用的协议。

如果是新建的项目，推荐在一开始就使用 `git@github.com:{USER}/{PROJECT}.git` 进行 `clone`。这样默认都是用 `ssh` 了。

如果是已有项目，使用 `git remote set-url {repository} {url}` 更改。

```bash
$ git remote -v
origin  https://github.com/abc/bcd.git (fetch)
origin  https://github.com/abc/bcd.git (push)

$ git remote set-url origin git@github.com:abc/bcd.git

$ git remote -v
origin  git@github.com:abc/bcd.git (fetch)
origin  git@github.com:abc/bcd.git (push)
```

## 强制覆盖本地文件

啥都别说了，直接重来吧：

```bash
git fetch --all
git reset --hard origin/master
```

## 撤销修改

### add 之前撤销

```bash
# 单个文件
git checkout FileName

# 所有文件
git checkout .
```

### commit 之前撤销

```bash
# 取消暂存
git reset HEAD FileName

# 撤销修改
git checkout FileName
```

### push 之前撤销

```bash
git reset [--hard|soft|mixed] [commit|HEAD]
```

## 删除历史提交记录

commit 多了项目也会膨胀... 清了一干二净。

```bash
# 用 orphan 参数创建全新的分支
git checkout --orphan {new_branch}

# 添加所有文件
git add -A

# 提交
git commit -am "commit message"

# 删除原始分支
git branch -D {old_branch}

# 交换分支
git branch -m {old_branch}

# 强制提交变更
git push -f origin {old_branch}
```

## 忽略文件权限

文件在系统间转移的时候可能由于权限改变而使文件全都处于 "modified" 状态。（比如从 linux 到 Windows，权限会从 644 变成 755）可以以项目为单位设置忽略文件权限：

```bash
git config core.filemode false
```
