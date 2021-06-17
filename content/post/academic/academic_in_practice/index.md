---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Academic 实现 Github Page 个人博客"
subtitle: ""
summary: ""
authors: []
tags: ['Hugo', 'Academic']
categories: ['Hugo', 'Academic']
date: 2018-12-09T16:34:42+08:00
lastmod: 2018-12-09T16:34:42+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Smart"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

{{% toc %}}

## Hugo 安装 / 更新

[Hugo](https://gohugo.io/) 是使用 `Go` 语言开发的静态站点生成器。不过无需准备 `Go` 语言环境，可以直接通过二进制编译包进行跨平台部署。

以下均以 Ubuntu18.0 为例。

### 安装 / 更新

1. 前往 [Github 页面](https://github.com/gohugoio/hugo/releases) 下载最新版本，这里我们下载 `hugo_0.52_Linux-64bit.deb`;
2. 使用命令 `dpkg -i hugo_0.52_Linux-64bit.deb` 安装 hugo;
3. 更新即重复上面两步，覆盖安装即可;

### 常用命令

- **`hugo`**： 编译项目生成静态网站，默认位置在项目的 `public` 目录下
- **`hugo server`**： 启动你的网站服务，可以通过浏览器访问 `http://127.0.0.1:1313/` 访问站点;
- **`hugo new {folder}/{name}.md`**: 创建新文章，使用 `markdown` 进行排版，一般默认放在 `post` 文件夹下；

基本没了，一般情况下用这三个命令就够了。

## Academic 安装 / 更新

Academic 是一个 Hugo 主题，从名字就可以知道这个主题比较学院派，适合科研 / 学术人员发布个人信息 / 介绍科研项目，当然，拿来做个人博客也是没问题的。

### 通过 Netlify

Academic 推荐使用第三方博客管理平台 [Netlify](https://app.netlify.com/start/deploy?repository=https://github.com/sourcethemes/academic-kickstart) 安装，如果你没有域名或者没想建站，只是想自己使用，那我建议不使用它的服务——请直接跳到下一部分，否则跟随网站引导完成安装;

### 通过 Git

#### 安装

1. 通过 git 安装的话，首先建议你在 GitHub 上 fork 成你自己的项目，默认的话，通过 `git clone https://github.com/sourcethemes/academic-kickstart.git My_Website` 将代码克隆到本地文件夹 `My_Website` _(当然，更推荐使用 ssh 协议，更安全，也免于 push 时输入密码，这里暂时按官方的来)_ ;
2. 进入文件夹，初始化项目：`git submodule update --init --recursive`，完成安装;

#### 自动更新

说是自动，还是需要手动执行一条命令：`git submodule update --remote --merge`;

这么做的前提条件是你是 `install` 的，也就是 `git submodule update --init --recursive` 过的，而不是直接把 `academic` 给 clone 到 themes 文件夹。

#### 手动更新

如果是 clone 到 themes 文件夹的话要这么更新：

1. `cd themes/academic`;
2. 将 `origin` 仓库重命名为 `upstream`：`git remote rename origin upstream`;
3. 将更新下载到本地：`git fetch upstream`;
4. 列出可用更新：`git log --pretty=oneline --abbrev-commit --decorate HEAD..upstream/master`;
5. 更新：`git pull upstream`;

## 部署到 Github Pages

### 原理

网上介绍的办法很多，但核心其实就一句：

**将 `hugo` 命令生成的 `public` 文件夹上传到 GitHub pages 项目下**。

`public` 文件夹相当于编译完成的静态网站，你在本地打开其实就能看。换句话说，你每次手动将这个目录下的内容上传到你的 GitHub page 项目也是可以的。

然后为了达到这个目的，Academic 给出的做法是利用 `git submodule` 将你的 `GitHub page` 项目作为 `My_Website` 项目的子模块存放到 `public` 目录。那么当你更新你的文章之后，只提交 `public` 文件夹内的变更到 `GitHub page` 项目即可。

### 官方教程 (缩减版)

_[原教程](https://sourcethemes.com/academic/docs/deployment/#github-pages) 看这里；_

1. 在 GitHub 上创建两个项目，一个是 fork 的 [academic-kickstart](https://github.com/sourcethemes/academic-kickstart.git)，也就是你前面 clone 到本地的 `My_Website`，另一个即是以你用户名 / 组织名开头、以 `.github.io` 结尾的 GitHub page 项目。
2. 在 `My_Website` 目录下执行 `git submodule update --init --recursive` 将子模块更新到最新状态；
3. 将 `config.toml` 中的 `baseurl` 设置为你的 GitHub page 地址；
4. **(实质)** 删除 `public` 文件夹 (如果有的话)，将 GitHub page 项目添加为子模块：`git submodule add -f -b master https://github.com/<USERNAME>/<USERNAME>.github.io.git public`;

    > _这时候你的 `My_Website` 项目实际上有两个子模块：作为主题依赖的 `themes/academic` 和作为网站的 `<USERNAME>.github.io`；_
    >
    > _有意思的是一般是子模块 `themes/academic` 更新了之后，你更新主项目 `My_Website` 的依赖；_
    >
    > _而你更新主项目 `My_Website` 的文章之后，再会手动的更新子模块 `<USERNAME>.github.io`，刚好反过来。_

5. 新增 / 编辑文章后，更新 `academic-kickstart` 项目：

    ```bash
    git add .
    git commit -m "Initial commit"
    git push -u origin master
    ```

6. 更新 GitHub page 项目：

    ```bash
    hugo
    cd public
    git add .
    git commit -m "Build website"
    git push origin master
    ```

实际上只有第六步是更新 GitHub page，每次重复执行这一部分就行 (如果你不把文章保存到 `academic-kickstart` 的话)。

### 脚本

`Hugo` 官方把上面步骤打包到了一个脚本：

```bash
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [$# -eq 1]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..
```

实际上我们干脆连第五步也放进去呗：

```bash
#!/bin/bash
# Receive args.
if [$1 = "push"]; then
    if [$# -eq 1]; then
        TIME_NOW=$(date +%T\ %F)
        MSG="Change something nobody knows at ${TIME_NOW}..."
        EDITED_FILE="."

    elif [$# -eq 2]; then
        MSG="$2"
        EDITED_FILE="."

    elif [$# -gt 2]; then
        MSG="$2"
        shift 2
        EDITED_FILE="$*"

    else
        echo "WTF?"

    fi

    echo "\033[0;32m
    ---------------------------
    Deploying to GitHub Page...
    ---------------------------
    \033[0m"

    # Build the project.
    hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

    # Go To Public folder
    cd public

    # Add changes to git.
    git add .

    # Commit changes.
    git commit -m "$MSG"

    # Push source and build repos.
    git push origin master

    # Come Back up to the Project Root
    cd ..

    echo "\033[0;32m
    -----------------------------
    Updating content to GitHub...
    -----------------------------
    \033[0m"

    # Add changes to git.
    git add $EDITED_FILE

    # Commit changes.
    git commit -m "$MSG"

    # Push source and build repos.
    git push origin master

elif [$1 = "pull"]; then

    # Update main repo.
    git pull

    # Update submodule.
    git submodule update

    echo "Synchronize finish."

else
    echo "Determine What you wanna do."

fi

```

将脚本保存为 `deploy.sh`，放到项目根目录下，完成修改后执行 `./deploy.sh` + `pull`/`push` 一键从服务器同步 / 提交 + 部署。

参数的第一个是你要执行的动作，从远程服务器 down 到本地的话就是 `./deploy.sh pull`，不用接别的。

如果是要将更新上传到服务器并部署，那就执行 `./deploy.sh push` + commit message，提交消息可以不写 (但最好还是写一下)：

`./deploy.sh push "{Your optional commit message}"`。

如果修改了多个文件，只想提交其中的一部分文件以保持 commit 的纯净，那就在 mesaage 后面附加你要提交的文件路径 (不超过 10 个...)：

`./deploy.sh push "{Your optional commit message}" path1 path2...`。

## 个性化配置

项目目录结构大体如下：

- `content 目录`： 网站内容，`home` 是你的主页的小控件，`post` 是默认文章存放位置
- `public 目录`： 生成的静态页面
- `resouces 目录`： JS 资源存放位置
- `static 目录`： 静态资源存放位置
- `themes 目录`： 主题文件所在目录
- `config.toml`: 全局配置文件

### config.toml 主要配置项解释

| 配置项                         | 说明                                                                                                                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| baseurl                        | 你的站点的 url，不设置这个你的文章 / 资源可能相互引用不到                                                                                                                                   |
| title                          | 网站标题                                                                                                                                                                                    |
| defaultContentLanguage         | 默认语言，中文的话填 `zh`，在文件末尾还有一处配置要同时修改才行                                                                                                                             |
| hasCJKLanguage                 | 是否有中 / 日 / 韩语                                                                                                                                                                        |
| defaultContentLanguageInSubdir | 目录是否允许用默认语言，`true` 就对了                                                                                                                                                       |
| highlight_languages            | 语法高亮，支持的语言可以去 [highlight.js](https://cdnjs.com/libraries/highlight.js/) 查到                                                                                                   |
| [[menu.main]]                  | 这部分是你主页上标题栏显示的内容，`url` 默认和你 `content/home` 下的文件名对应                                                                                                              |
| Languages                      | 添加中文支持的话，把 `[languages.zh]` 部分解除注释，`languageCode` 写 `"zh-cn"`，添加其他语种的话，相同格式再写一组 `[languages.XX]` 即可，支持的语言代码可以在 `themes\academic\i18n` 查看 |

### 修改网站 logo

默认的 logo 是 Academic 的蓝色学位帽，想替换的话将你想用的 logo 保存为
`icon.png`(默认 32*32 像素，大了也没关系)
和 `icon-192.png`(192*192 像素)，并放到项目的 `static/img` 目录下

### 给文章添加精选图

这个图片只能添加一个，名字必须是 `featured.*`(后缀 jpg/png 都行)，而且必须和文章放在同一个文件夹下。

所以一般做法是把文章 `aaa.md` 改名为 `index.md` 并新建一个 `aaa` 目录，再和 `featured.png` 图片一起扔进去。

显示的效果是在文章列表页，文章右侧有一个缩略图；打开文章，标题默认会居左，右边是精选图：

![example](https://i.loli.net/2021/06/17/w9J1YNL7zDE26od.png)

### 给文章添加头部背景

这个是文章头部的横跨整个页面的大图，也就是文章头部这个黑底白字的大图。

这个的图片可以放到 `static/img` 目录下，不过需要在你文件的 `+++` 的部分添加如下代码：

```markdown
[header]
  image = "img 名称"
  caption = "标题说明"
```

顺便一提，文章内引用 `static/img` 下存储的图像的话，路径大致如此 `![example](/media/image_abc.png)`

### 目录

使用 `{``{% toc %}}` 加在文章的任何你想要的地方以自动生成目录

### 注意 / 警告标识

被 `{``{% alert note %}}` 和 `{``{% /alert %}}` 包裹起来的内容即为注意项：

{{% callout note %}}
注意内容 blabla
{{% /callout %}}

被 `{``{% alert warning %}}` 和 `{``{% /alert %}}` 包裹起来的内容即为警告项：

{{% callout warning %}}
警告内容 blabla
{{% /callout %}}

### 消除短代码效果

Hugo 是基于 Go 的 Template，所以所有以 `{``{% %}}` 或者 `{``{< >}}` 包裹的内容都会被解析为短代码块，而无法直接显示其代码。
那么我是怎么解决的呢，分情况：

#### 单行代码

你看到的 `{``{% toc %}}` 实际是由 `` `{` `` 和 `` `{% toc %}}` `` 组成的。

而上面打出的 `` ` `` 其实是 ` 双反引号 空格 反引号 空格 双反引号 `，就不再嵌套了。。。

#### 代码块

```go
{{</* figure library="1" src="image.jpg" title="A caption" */>}}
```

的本质是将 <> 或者 %% 内的内容用 /**/ 注释掉：

`{``{</* figure library="1" src="image.jpg" title="A caption" */>}}`

{{% callout note %}}
谁知道 Markdown 的代码块怎么嵌套...
比如外层一个 markdown 的代码块，
里面要显示包含反引号格式的 python 代码块...
{{% /callout %}}

### 修改模板

比如你看到我每个文章结尾都有一个 [`CC4.0 协议`](http://creativecommons.org/licenses/by-sa/4.0/) 的标志，这个肯定不是一篇篇手动添加的，实际上我是自己写了一个 License 的 Widget，插入到文章的模板里面实现的。

{{% callout warning %}}
不要直接修改 `theme` 里面的内容，否则更新主题的时候会非常尴尬。
{{% /callout %}}

正确的做法是在项目根目录建立 `layouts` 文件夹，将你想修改的模板从 `themes/academic/layouts` 拷贝过来再修改。

现在 Academic 主题的 layouts 大概是这样的：

- `_default`: 默认文章相关模板；
- `docs`: 文档相关模板;
- `home`: 主页相关模板;
- `partials`: 小部件相关模板，页面头部 / 脚注 / 摘要等等的都在这;
  - 额外的有一个 `widgets` 文件夹，里面是主页的 widget 的模板；
- `project`: 项目相关模板;
- `publication`: 出版物相关模板;
- `section`: 摘要相关模板;
- `shortcodes`: Academic 提供的额外效果模板，你写的所有 `{``{%%}}` 的内容效果都出自这里;
- `slides`: 幻灯片相关模板;
- `talk`: 宣讲相关模板;

继续用我自己做例子，我新建了 `layouts/partials/license.html`，把 CC 协议相关内容存了进去，
接着，复制主题目录下的 `layouts/_default/single.html` 到对应位置，在合适地方插入一句 `{{partial "license.html" .}}`，表明我要在这里使用名为 `license.html` 的 `partial`。
再之后就是你们看到的效果了。
