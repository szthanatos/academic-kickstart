---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "(翻译)Academic 文档 - 内容管理"
subtitle: ""
summary: ""
authors: []
tags: ['Translation', 'Academic', 'Hugo']
categories: ['Academic', 'Hugo']
date: 2019-01-01T12:54:11+08:00
lastmod: 2019-01-01T12:54:11+08:00
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

{{% callout note %}}
本文是对 [Academic 文档 - Managing content 章节](https://sourcethemes.com/academic/docs/managing-content/) 的个人翻译，基于个人理解，不保证绝对准确。

原文见上方连接。
{{% /callout %}}

这是一个使用 Academic 框架管理你的文章的简短指南。
Academic 提供的内容模板包括出版物、项目、宣讲、新闻 / 博客文章、以及小部件页。
之后，你可能同样对 [使用 Markdown、LaTeX 数学公式和代码段进行创作](/post/trans_writing_content.md) 感兴趣。

{{% callout warning %}}
Hugo V0.49 版本在使用本指南中的 `hugo new` 命令时存在一个 bug，请升级到 V0.50 及以上。
{{% /callout %}}

## 精选图片

要在文章页显示一个精选图片，简单的将名为 `featured.*`(e.g. `featured.jpg`) 的图片文件拖拽到文章文件夹即可。

{{% callout note %}}
如果你的页面在它所属的分类文件夹下没有自己的文件夹 ([页面包](https://gohugo.io/content-management/page-bundles/))，
你可以创建一个和你页面 `NAME.md` 同名的文件夹 `NAME`，并将页面文件放入文件夹中，变为 `NAME/index.md`。
这里有一个 [自动迁移工具](https://github.com/sourcethemes/academic-scripts)。
使用页面包需要 Academic v3+ 以及 Hugo v0.50 以上。
{{% /callout %}}

想要为图片添加标题或者设置一个焦点以控制图片的裁剪？
将下方的参数添加到扉页 (也就是 md 文件 `+++` 括起来的部分) 的底部以自定义图片的外观。
标题 (caption) 参数支持使用 Markdown 为图片添加标题或描述。
焦点 (focal_point) 参数确保图片自动缩放的时候主要内容始终可见。

```markdown
# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
[image]
  # Caption (optional)
  caption = "Photo by [Academic](https://sourcethemes.com/academic/)"

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = "Smart"

  # Show image only in page previews?
  preview_only = false
```

## 标题图片

将下面的 header 参数添加到扉页的末尾，以在页面顶部展示一个占据全部宽度的标题图片。
图片文件默认会从静态图片文件库 `static/img/` 读取 (所以不必写全)，所以下面例子的图片文件的完整路径是 `static/img/header.png`。
标题参数作用和精选图一致。

```markdown
[header]
  image = "header.png"
  caption = "Image credit: [**Academic**](https://github.com/gcushen/hugo-academic/)"
```

## 数学公式和代码

要在页面中启用 **LaTeX** 渲染数学公式，在页面的扉页中应该申明 `math = true`，如同示例网站的例子一样。
或者，在 `config.toml` 中设置 `math = true`，以在全局范围内允许数学公式渲染。

在 `config.toml` 中设置 `highlight = false` 以全局禁用代码高亮。你可以在需要代码高亮的页面的扉页单独设置 `highlight = true`。
查看 [code-highlighting docs](https://sourcethemes.com/academic/docs/writing-markdown-latex/#code-highlighting) 以获取更多细节。

## 页面特性

将下述参数添加到页面扉页以管理页面特性：

```markdown
reading_time = false  # 显示估计阅读时间
share = false  # 显示分享按钮
profile = false  # 显示作者信息
comments = false  # 显示评论区
```

## 创建一个出版物

### 自动

先进的文献管理工具可以帮助你将你的出版物转化为开源的 BibTeX 格式。
如果你是新手的话我们推荐你使用流行的开源工具 [Zotero](https://www.zotero.org/) 来管理你的文献。

在你的文献管理工具中创建你自己的出版物列表并导出为 `*.bib` 格式的 BibTeX 文件。

工具需要 Python3 环境，所以请先 [安装 Python3](https://realpython.com/installing-python/)。
同样，为了让你有机会检查 Academic 管理工具产生的改变，你需要备份你的站点，或者确保它已经处于 Git 的管理之中。

打开你的终端或者命令提示符应用，安装 Academic 管理工具：

```bash
pip3 install -U academic
```

使用 `cd` 命令进入你的站点目录。

之后，导入你的出版物：

```bash
academic import --bibtex <path_to_your/publications.bib>
```

这个工具尚处于测试阶段，目的是为了给你提供辅助。
所以在发布你的站点之前你应该检查 `publications` 文件夹下产生的内容。
你同样可以看看下一章节 ` 手动 ` 部分有关扉页参数的细节，以想办法增强展示效果。

想要支持这个工具或者提供建议 / 反馈，请查看 [Academic admin tool 项目主页](https://github.com/sourcethemes/academic-admin)。

### 手动

另一种选择，使用命令手动的创建出版物：

```bash
hugo new --kind publication publication/<my-publication>
```

`<my-publication>` 是你得出版物的名称，使用 `-` 代替空格。

之后，编辑 `content/publication/<my-publication>/index.md` 内含有你的出版物信息的参数。主要参数如下：

- **title**: 标题
- **date**: 发布日期 (必须使用有效的 TOML 日期格式)
- **publication_types**: 使用图例来说明你出版物的类型, e.g. conference proceedings
- **publication**: 你的出版物发布在什么地方 - 允许使用 Markdown 以标注斜体或其他.
- **abstract**: 摘要

使用 Markdown 格式将你出版物的细节写到文档的正文部分 (在 `+++` 部分之后)。
内容会出现在你的出版物页之上。

要使访客能够阅读到你的作品，将作品的 PDF 链接填入 `url_pdf`，或者将作品 PDF 文件放置到出版物目录并统一为相同命名，这样会自动生成 PDF 的链接。
举例，如果你的出版物说明位于 `publication/photons/index.md`，将 PDF 文件重命名并放到 `publication/photons/photons.pdf`。

### 关联其它资源

使用 `url_` 链接以指向本地 / 网络内容。
要关联本地内容的话将之复制到出版物文件夹并使用例如 `url_code = "code.zip"` 的方式添加引用。

你也可以将下面的代码块添加到扉页，以使用自定义链接按钮：

```bash
url_custom = [{name = "Custom Link 1", url = "http://example.org"},
              {name = "Custom Link 2", url = "http://example.org"}]
```

{{% callout warning%}}
想要在扉页的参数中使用双引号或者反斜杠需要额外添加一个反斜杠，_例子懒得翻_，更多信息请参阅 [TOML 文档](https://github.com/toml-lang/toml#user-content-string)。
{{% /callout %}}

## 创建博文

要创建一篇新文章：

```bash
hugo new  --kind post post/my-article-name
```

然后用你的完整标题和内容填充新生成的 `content/post/my-article-name.md` 文件。

Academic 会自动生成内容摘要并显示在主页上。
如果你不满意自动生成的摘要内容，你可以在文章内容中放置 `<!``--more-->` 以限定摘要的长度，
或者像这样：

```bash
summary = "Summary of my post."
```

在扉页内添加 `summary` 参数以覆盖自动生成的摘要。

要为特定的文章禁止评论，在扉页添加 `disable_comments = true` 参数。
要全局的禁止评论的话，在 `config.toml` 中设置 `disqusShortname = ""` 或者 `disable_comments = true`。

## 创建项目

要创建一个项目：

```bash
hugo new  --kind project project/my-project-name
```

之后编辑新生成的 `content/project/my-project-name.md` 文件。
在扉页将 `external_link = "http://external-project.com"` 设置成已经存在的项目网址，
或者也可以手动在正文中介绍项目的情况。

## 创建演讲

要创建一个演讲：

```bash
hugo new  --kind talk talk/my-talk-name
```

然后用你的完整标题和内容填充新生成的 `content/talk/my-talk-name.md` 文件。
你会注意到演讲的很多参数和出版物是类似的。

## 创建幻灯片

可以使用 Markdown 非常高效的创建幻灯片并通过你的网站分享给观众。
甚至还包括演讲者笔记。

查看 [slides demo](https://themes.gohugo.io//theme/academic/slides/example-slides#/)
——尽管你可以注意到这个幻灯片是由 Hugo 团队制作的，并且他们缩减了一些功能。
运行 `themes/academic/exampleSite/` 下的示例站点以查看完整的包含演讲者笔记的示例。

查看 `themes/academic/exampleSite/content/slides/example-slides.md` 内的 `example slide deck` 以开始学习。

在演讲 / 出版物页面使用 `url_slides` 参数来关联到幻灯片。
比如，`url_slides = "slides/example-slides"` 可以关联到上面的示例站点。
在 [这里](https://raw.githubusercontent.com/gcushen/hugo-academic/master/exampleSite/content/talk/example/index.md) 可以看到包含 `url_slides` 的完整扉页的示例。

## 创建课程或文档

_文档_ 是用来 **分享知识** 的。常见例子包括在线课程、教程、软件文档以及知识库。

你现在阅读的这个页面 (不是我翻译之后的这个) 就是用_文档_的方式来展现 Academic 相关的。
同样，这里也有一个 [在线课程](https://themes.gohugo.io//theme/academic/tutorial/) 的例子。

查看 `themes/academic/exampleSite/content/tutorial/` 的示例以学习如何开始。

如果你是一名使用 R 语言的数据分析师 / 数据科学家 (e.g. RStudio and RMarkdown)，我们推荐你阅读 [R boilerplate project on GitHub](https://github.com/sourcethemes/project-kickstart-r)。

## 创建小部件页面

你是否想利用 Academic 的小部件系统，创建一个和 Academic 主页类似的页面？

在你的 `content` 文件夹下创建一个新的，以你的页面命名的文件夹。在这个例子中我们将创建 `content/tutorials/` 文件夹以创建我们的 tutorials 页。

在新建的 `content/tutorials/` 文件夹下创建一个名为 `_index.md` 的文件，内容如下：

```bash
+++
title = "Tutorials"  # Add a page title.
date = 2017-01-01T00:00:00  # Add today's date.
widgets = true  # Page type is a Widget Page.
summary = ""  # Add a page description.
+++
```

将你的小部件放入 `content/tutorials/` 文件夹，可以通过复制 `content/home/` 下的小部件或者从 [Github](https://github.com/gcushen/hugo-academic/tree/master/exampleSite/content/home) 上下载来实现。

## 创建其他页面 (e.g. 简历)

其他类型内容的话，可以创建自己的自定义页面。
例如，我们在 `content` 文件夹下创建一个 `cv.md` 简历页面。复制任意一个文章的扉页，根据需要进行调整，然后在下面编辑 Markdown 内容。
再之后，您可以使用 `[My CV]{``{< ref "cv.md" >}}` 代码将简历添加到任何现有页面的内容上。

或者，在上面的例子中，我们可以使用简历的 PDF 文件。为此，在 `static` 文件夹中创建名为 `files` 的文件夹，并将名为 `cv.pdf` 的 PDF 文件移动到该位置。
然后可以使用以下代码将 PDF 文件链接到你的任意内容中：`{``{% staticref "files/cv.pdf" %}}`` 下载我的简历 ``{``{% /staticref %}}`。

## 管理列表页

档案 (archive，我理解的就是列表) 页或者说节点页，是列出你所有内容的特殊页面。
博客文章、出版物、演讲都会有列表页。
如果存在一个小部件放不下的内容的话，主页上的小部件会自动链接到列表页。
因此，如果你没有足够多的内容的话你可能不会看到自动生成的链接——
不过你也可以在文章中用一般的 Markdown 链接格式，手动的链接他们。

你可以通过将以下 `_index.md` 文件从示例站点复制到你的 `content/` 文件夹中的相同位置，来编辑标题并添加自己的内容（比如简介）：

```bash
/themes/academic/exampleSite/content/post/_index.md
/themes/academic/exampleSite/content/publication/_index.md
/themes/academic/exampleSite/content/talk/_index.md
```

之后，根据需要编辑每个 `_index.md` 中的 `title` 参数，并在扉页之后添加任何内容。
你可能注意到 `_index.md` 文件略有不同，其中一些具有可用于关联内容类型的特殊选项。
例如，`publication/_index.md` 包含用于设置出版物列表页面上显示的列表的引用样式的选项。

## 移除内容

通常来说，要移除任意内容，简单的从你的 `content/post`、`content/publication`、`content/project` 或者 `content/talk` 文件夹中删除对应页面文件即可。

## 查看站点更新

在你对站点做出修改之后，你可以通过执行 `hugo server` 并在浏览器中打开 <http://localhost:1313/> 来看到效果。

## 部署站点

最后，你可以 [部署你的站点](https://sourcethemes.com/academic/docs/deployment/) 了。
