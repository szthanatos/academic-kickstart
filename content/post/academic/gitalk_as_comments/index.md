---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "使用 Gitalk 作为 Academic 的评论插件"
subtitle: ""
summary: ""
authors: []
tags: ["Academic", "Hugo", "Gitalk"]
categories: ["Academic", "Hugo", "Gitalk"]
date: 2020-06-17T23:21:08+08:00
lastmod: 2020-06-17T23:21:08+08:00
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

Academic(Wowchemy) 作为我 GitHub Pages 博客主题，默认提供3种类型的评论(comments)插件：

- Twitter——需要跳转到外部页面
- Disqus——收费 or 广告
- Commento——收费 or 自建

总之，都不完美。

所以我这边使用 [Gitalk](https://gitalk.github.io/) ——一个基于 GitHub Issues 的开源评论系统，提供评论功能。

具体替换步骤如下：

## 0. 开启评论功能

在项目 `config/_default/params.yaml` 文件中，评论相关配置如下：

```yaml
comments:
  provider: ''
  commentable:
    post: true
    book: true
    project: true
    publication: true
    event: true
```

## 1. 获取 GitHub Application 密钥对

点击链接 [创建 GitHub Application](https://github.com/settings/applications/new)

![创建 GitHub Application](https://i.loli.net/2021/06/17/vqWohkOplCwIx7N.png)

保存好 `Client ID` 和 `Client Secret` 以待下一步使用。

## 2. 创建 Gitalk 插件

创建 `gitalk.html` 如下，放置到项目 `layouts/partials/comments/` 目录下：

```html
<div id="gitalk-container"></div>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/blueimp-md5@2.18.0/js/md5.min.js"></script>
<script>
    var gitalk = new Gitalk({
        clientID: '【你的 Client ID】',
        clientSecret: '【你的 Client Secret】',
        repo: '【你的博客地址】',
        owner: '{{ .Site.Params.Gitalk.owner }}',   // GitHub repo 所有者
        admin: ['{{ .Site.Params.Gitalk.owner }}'], // GitHub repo 所有者和合作者，对这个 repo 有写权限
        id: md5(location.pathname),                 // 页面唯一标识，以 GitHub issue tag 形式存在
        distractionFreeMode: false                  // 评论框的全屏遮罩效果
    })
    gitalk.render('gitalk-container')
</script>
```

{{% callout note %}}
注意这里使用的是 [jsdelivr](https://cdn.jsdelivr.net) 提供的 CDN 服务，你也可以选择更适合你情况的供应商。
{{% /callout %}}

{{% callout note %}}
gitalk id 被设置为 MD5 化的 URL 的路径，这是因为 GitHub tag 限制长度不能超过50，如果路径过长会无法正确创建 issue；

不过这样会让 issue tag 都是一串串 MD5 码...
{{% /callout %}}

## 3. 覆盖原生评论插件

原生 [academic 主题](https://github.com/wowchemy/wowchemy-hugo-modules/)控制使用哪种评论的文件是
`wowchemy-hugo-modules/wowchemy/layouts/partials/comments.html`。

我们在项目路径下直接创建 `layouts/partials/comments.html` 覆盖它的效果：

```html
{{ if index site.Params.comments.commentable .Type | and (ne .Params.commentable false) | or .Params.commentable }}
<section id="comments">
    {{ partial "comments/gitalk" . }}
</section>
{{ end }}
```

这样只要是能评论的页面，就都会在底部出现评论框了。

## 4. 初始化 issue

配置完成后，本地启动验证一下，出现下图效果就说明已经基本成功了。

![本地效果](https://i.loli.net/2021/06/18/dbYk1IVvtjDqazP.jpg)

当然，你现在点开 `使用 GitHub 登录` 是没用的，因为项目在你本地。

Gitalk 不会自动创建 issue，手动初始化的方式就是部署完，在页面登录 GitHub 然后挨个点开你所有文章。

自动化方式可以网上搜一下相关教程，基本都需要 node 环境，我这边打算写个 Python 的...立个 Flag `_(:з」∠)_`
