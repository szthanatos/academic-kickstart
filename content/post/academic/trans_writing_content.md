---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "(翻译)Academic 文档 - 内容编写"
subtitle: ""
summary: ""
authors: []
tags: ['Translation', 'Academic', 'Hugo']
categories: ['Academic', 'Hugo']
date: 2019-01-01T12:54:29+08:00
lastmod: 2019-01-01T12:54:29+08:00
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
本文是对 [Academic 文档 - Writing content 章节](https://sourcethemes.com/academic/docs/writing-markdown-latex/) 的个人翻译，基于个人理解，不保证绝对准确。

原文见上方连接。
{{% /callout %}}

Academic 支持使用 Markdown、LaTeX 数学公式和 Hugo 代码段编写内容。
此外，可以使用 HTML 以实现高级样式。
本文概述最常见的格式选项。

## 副标题

```bash
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6
```

## 强调

```bash
# 下划线内为斜体
Italics with _underscores_.

# * 内为粗体
Bold with **asterisks**.

# 粗体和斜体可以组合
Combined emphasis with **asterisks and _underscores_**.

# 双波浪符内为删除线
Strikethrough with ~~two tildes~~.
```

## 有序列表

```markdown
1. First item
2. Another item
```

## 无序列表

```markdown
我个人更习惯用 -
* First item
* Another item
```

## 图片

图片可以存放在你的媒体库 `static/img` 或你的 [页面文件夹](https://gohugo.io/content-management/page-bundles/)。
使用以下任一方式即可引用图片：

假设图片来自你的 `static/img` 媒体库：

```go
{{</* figure library="1" src="image.jpg" title="A caption" */>}}
```

假设图片来自你的页面文件夹 (比如 `content/post/hello/`)

```go
{{</* figure src="image.jpg" title="A caption" */>}}
```

带编号和标题的图片：

```go
{{</* figure src="image.jpg" title="A caption" numbered="true" */>}}
```

一般图片：

```markdown
![alternative text for search engines](/media/image.jpg)
```

## 图片集

为页面包增加一个图片集：

1. 在 [页面包](https://gohugo.io/content-management/page-bundles/)(也就是你的页面文件夹) 内创建图片集文件夹；
2. 将图片放入图片集文件夹；
3. 粘贴 `{``{< gallery album="<ALBUM FOLDER>" >}``}` 到文章中你想要它出现的地方，将 album 参数修改为你文件集的名称；

可选的，要为你的图片集添加标题的话，将下面的实例添加到你扉页的尾部:

```bash
[[gallery_item]]
album = "<ALBUM FOLDER>"
image = "<IMAGE NAME>.jpg"
caption = "Write your image caption here"
```

另外，想要在图片集中使用网络位置 / 媒体库中的图片；

1. 将图片添加到 `static/img/` 文件夹；
2. 在文章的扉页尾部声明图片引用：

    ```bash
    [[gallery_item]]
    album = "1"
    image = "my_image.jpg"
    caption = "Write your image caption here"

    [[gallery_item]]
    album = "1"
    image = "https://raw.githubusercontent.com/gcushen/hugo-academic/master/images/theme-dark.png"
    caption = "Dark theme"
    ```

3. 在正文要显示的位置使用 `{``{< gallery album="1">}``}`

## 视频

页面可以添加以下几种类型的视频。

### 本地视频文件

要添加视频，将它放在 `static/img/` 媒体库或者页面文件夹内，使用以下任一方式即可引用。

位于 `static/img/` 文件夹下的视频：

```go
{{</* video library="1" src="my_video.mp4" controls="yes" */>}}
```

位于页面文件夹下的视频：

```go
{{</* video src="my_video.mp4" controls="yes" */>}}
```

### Youtube

```go
{{</* youtube w7Ft2ymGmfc */>}}
```

### Vimeo

```go
{{</* vimeo 146022717 */>}}
```

## 链接

```markdown
[I'm a link](https://www.google.com)
[A post]({{</* ref "post/hi.md" */>}})
[A publication]({{</* ref "publication/hi.md" */>}})
[A project]({{</* ref "project/hi.md" */>}})
[Another section]({{</* relref "hi.md#who" */>}})
```

想要链接到一个文件，比如 PDF，首先将它放到 `static/files/` 文件夹下，然后使用下面方式链接：

```go
{{%/* staticref "files/cv.pdf" "newtab" %}}Download my CV{{% /staticref */%}}
```

`staticref` 的 `"newtab"` 参数将使链接在新页面打开。

### 标签和分类

使用 `{``{< list_tags>}``}` 生成标签链接列表，使用 `{``{< list_categories >}``}` 生成分类链接列表。

## Emojis

可用 Emojis 见 [Emoji cheat sheet](http://www.webpagefx.com/tools/emoji-cheat-sheet/)。
下面的这个示例在实际使用时需要把: 和表情名之前的空格去掉：

```bash
I : heart : Academic : smile :
```

I :heart: Academic :smile:

## 段落引用

```bash
> This is a blockquote.
```

> This is a blockquote.

## 高亮引用

```go
This is a {{</* hl>}}highlighted quote{{< /hl */>}}.
```

This is a {{<hl>}}highlighted quote{{< /hl >}}.

## 脚注

```markdown
I have more [^1] to say.
[^1]: Footnote example.
```

## 嵌入文档

下面几种类型的文档可以被嵌入到页面中。

要插入 **谷歌文档** (比如幻灯片) 点击 Google Docs 中的 _File > Publish to web > Embed_ 并复制 `src="..."` 部分中的 URL。
之后粘贴到下面代码中：

```go
{{</* gdocs src="https://docs.google.com/..." */>}}
```

### Speaker Deck

```go
{{</* speakerdeck 4e8126e72d853c0060001f97 */>}}
```

## 代码高亮

将语言的代码，比如 `python`，作为参数放在三个反引号之后：(打出来 ``` 就会被解析，只能加空格了)

```html
 ` ` `python
 # Example of code highlighting
 input_string_var = input("Enter some data:")
 print("You entered: {}".format(input_string_var))
 ` ` `

```

效果：

```python
# Example of code highlighting
input_string_var = input("Enter some data:")
print("You entered: {}".format(input_string_var))
```

### 高亮选项

Academic 主题使用 [highlight.js](https://highlightjs.org/) 作为高亮的来源，并且默认为所有页面启用。
并且，有一些更细粒度的选项可以控制 highlight.js 的显示效果。

下表列出了 highlight.js 支持的一些选项，包含他们的类型和简短描述。
**config.toml** 列中的 "yes" 表示允许在 `config.toml` 中全局设置，
**preamble** 列中的 "yes" 表示可以设定在特定页面中。

| option              | type    | description     | config.toml | preamble |
| ------------------- | ------- | --------------- | ----------- | -------- |
| highlight           | boolean | 启用 / 禁用高亮 | yes         | yes      |
| highlight_languages | slice   | 选择额外语言    | yes         | yes      |
| highlight_style     | string  | 选择高亮样式    | yes         | no       |

#### `highlight` 选项

`highlight` 选项允许在全局或者特定页面启动 / 禁止语法高亮。
如果没有明确指定的话，默认会认为你设置了 `highlight = true`。
也就是说，highlight.js 的 javascript/css 文件会出现在每一个页面文件中。
如果你只希望那些真的需要使用的页面才有语法高亮，
你可以在 `config.toml` 中设置 `highlight = false`，
之后在需要的页面的扉页覆盖为 `highlight = true`。
相反，你也可以全局启用语法高亮，在不需要的页面中禁用。
下面给出一张展示不同全局和单独页面设置下，页面是否高亮。

| config.toml   | page preamble  | highlighting enabled for page? |
| ------------- | -------------- | ------------------------------ |
| unset or true | unset or true  | yes                            |
| unset or true | false          | no                             |
| false         | unset or false | no                             |
| false         | true           | yes                            |

#### `highlight_languages` 选项

`highlight_languages` 选项允许你指定 highlight.js 支持的，但是不是默认支持的常见的语言。
比如，你想在所有页面高亮 Go 和 clojure 语言，那就在 `config.toml` 中设置 `highlight_languages = ["go", "clojure"]`。
另外，如果你想为页面只启用特定的语法高亮，那就去页面扉页设置 `highlight_languages`。

在 `config.toml` 和扉页设置的 `highlight_languages` 是累加的。
也就是说，如果 `config.toml` 里设置了 `highlight_languages = ["go"]`，而扉页设置了 `highlight_languages = ["ocaml"]`，
那么这个页面会包含两者的高亮文件。

当你设置了 `highlight_languages` 之后，相应的高亮脚本会由 [cdnjs 服务](https://cdnjs.com/libraries/highlight.js/) 提供。
要查看支持的语言，访问 [cdnjs page](https://cdnjs.com/libraries/highlight.js/) 页面并查找包含 "languages" 关键字的链接。

`highlight_languages` 选项通过 CDN 提供了一种方便又容易的方式来满足附加语言的高亮需求。
如果 cdnjs 提供的默认的文件不能满足你的需求，你可以通过 [个性化指南](https://sourcethemes.com/academic/docs/customization/#add-scripts-js) 中的方法来使用自己的 javascript 文件。

#### `highlight_style` 选项

`highlight_style` 选项允许你使用备选的高亮样式。
比如，如果你想使用 solarized-dark 样式，你可以在 `config.toml` 中设置 `highlight_style = "solarized-dark"`。

如果未设置 `highlight_style`，默认会使用 Academic 提供的或者在你的 `static` 文件夹下的 `/css/highlight.min.css`。
Academic 提供的默认样式和 `github` 是一致的。

如果设置了 `highlight_style`，`/css/highlight.min.css` 就会被忽略，相应的样式会由 [cdnjs 服务](https://cdnjs.com/libraries/highlight.js/) 提供。
要查看支持的样式列表，访问 [cdnjs page](https://cdnjs.com/libraries/highlight.js/) 页面并查找包含 "styles" 关键字的链接。

可以在 [highlight.js demo page](https://highlightjs.org/static/demo/) 上查看可用样式。

{{% callout note %}}
不是所有 [highlight.js demo page](https://highlightjs.org/static/demo/) 上列出的样式都在 [cdnjs 服务](https://cdnjs.com/libraries/highlight.js/) 上可用。
如果你想使用不是由 cdnjs 提供的样式，那么保持 `highlight_style` 未设置，然后将相应文件放到 `/static/css/highlight.min.css`。
{{% /callout %}}

{{% callout note %}}
如果你不想更换 Academic 附带的样式，但是还是想由 [cdnjs](https://cdnjs.com/libraries/highlight.js/) 提供服务，那么在 `config.toml` 中设置 `highlight_style = "github"`。
{{% /callout %}}

只有在 `config.toml` 中设置的 `highlight_style` 才会生效，在扉页设置的 `highlight_style` 不会生效。

## Twitter tweet

```go
{{</* tweet 666616452582129664 */>}}
```

## GitHub gist

```go
{{</* gist USERNAME GIST-ID */>}}
```

## LATEX 数学公式

```go
$$\left [– \frac{\hbar^2}{2 m} \frac{\partial^2}{\partial x^2} + V \right ] \Psi = i \hbar \frac{\partial}{\partial t} \Psi$$
```

<div>
$$\left [– \frac{\hbar^2}{2 m} \frac{\partial^2}{\partial x^2} + V \right ] \Psi = i \hbar \frac{\partial}{\partial t} \Psi$$
</div>

另外，单行的数学公式可以只用单个 `$` 包裹：

```go
This is inline: $\mathbf{y} = \mathbf{X}\boldsymbol\beta + \boldsymbol\varepsilon$
```

This is inline: $\mathbf{y} = \mathbf{X}\boldsymbol\beta + \boldsymbol\varepsilon$

注意 Markdown 的特殊符号需要使用反斜杠转义，才能被识别为数学公式而非 Markdown 关键字。
比如 `*` 和 `_` 应该被替换为 `\*` 和 `\_`。

### 多行方程式

标准 LaTeX 的双反斜杠换行应该被替换为 6 个反斜杠：

```go
$$f(k;p\_0^\*) = \begin{cases} p\_0^\* & \text{if }k=1, \\\\\\
1-p\_0^\* & \text {if}k=0.\end{cases}$$
```

$$f(k;p\_0^\*) = \begin{cases} p\_0^\* & \text{if }k=1, \\\\\\
1-p\_0^\* & \text {if}k=0.\end{cases}$$

### 论文摘要

由于 Hugo 和 Academic 会尝试解析摘要中的 TOML, Markdown, 以及 LaTeX 内容，论文的 `abstract` 和 `abstract_short` 部分应当遵循下面两个方针：

- LaTeX 的反斜杠 `\` 应该被转义为双反斜杠，也就是 `\\`
- LaTeX 的下划线 `_` 应该被转义为双反斜杠加下划线，也就是 `\\_`

因此，`abstract = "${O(d_{\max})}$"` 就会变成 `abstract = "${O(d\\_{\\max})}$"`。

## 表格

代码：

```shell
| Command         | Description         |
| --------------- | ------------------- |
| `hugo`          | Build your website. |
| `hugo serve -w` | View your website.  |
```

效果：

| Command         | Description         |
| --------------- | ------------------- |
| `hugo`          | Build your website. |
| `hugo serve -w` | View your website.  |

## 警报

在你为文章添加提示、注意项、警告时，警报是一个非常有用的功能。
尤其是对于教程性质的文章。
使用对应的短代码以在文章中显示警报：

```cpp
{{%/* alert note */%}}
Here's a tip or note...
{{%/* /alert */%}}
```

这会显示为如下 _注意_ 项：

{{% callout note %}}
Here's a tip or note...
{{% /callout %}}

```cpp
{{%/* alert warning */%}}
Here's some important information...
{{%/* /alert */%}}
```

这会展示为如下 _警告_ 项：

{{% callout warning %}}
Here's some important information...
{{% /callout %}}

## 目录

目录对于长文章或者教程 / 文档可能特别有用，在你 Markdown 正文的任何位置使用 `{``{% toc %}``}` 短代码自动生成目录。

[^1]: Footnote example.
