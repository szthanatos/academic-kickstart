---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Tmux in 10 minutes"
subtitle: ""
summary: ""
authors: []
tags: ['Tmux']
categories: []
date: 2019-01-01T12:27:05+08:00
lastmod: 2019-01-01T12:27:05+08:00
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

## 简介

Tmux 是一个终端复用软件，默认的 Linux 终端一个会话只能干一件事，有了 tmux 就能在一个窗口同时管理多个前 / 后台程序了。

## 安装 tmux

基础软件，跳过

## 基础概念

见图

![Tmux 页面](https://i.loli.net/2021/06/17/Yc5AbRow7niOevt.jpg)

- `Session`：输入 tmux 后就创建了一个会话，一个会话是一组窗体的集合；
- `Window`：会话中一个可见的窗口；
- `Pane`: 一个窗口可以分成多个窗格；

用 win10 任务视图 (Win+Tab 调出) 的概念来类比，

`Pane` 就是一个个应用窗口，在一个桌面上可以同时开多个 (但是不能堆叠，)；

`Window` 就是一组组桌面，同一时间你只能看到一个桌面

`Session` 就是一个用户，区别就是 win10 同一个账户只能登陆一次，tmux 里相当于一个用户登陆 N 次。

为了控制这些元素，tmux 分为三种模式：

- ` 控制模式 `: （按下或者按住前缀 (tmux-prefix)，默认 ctrl+b, 下文用 `※ + X` 表示按下前缀之后按 X，`※※ + Y` 表示按住前缀的同时按 Y）相当于各种热键；
- ` 命令模式 `: （输入 tmux 后接命令，或者在 tmux 内输入 `※ + shift + :`）也就是输入命令，但是执行的不是系统命令，而是 tmux 自身的命令；
- ` 一般模式 `:  正常打字；

## 配置

和 zsh 一样，得先配置才能用的舒坦。下面是我个人用的配置文件。

`Ctrl+b` 被我替换为 `Ctrl+x`，横竖分割窗格我分别设置为 `-` 和 `\`，刚好一横一竖嘛，并且启用了 tpm 管理 tmux 插件。

```properties
#-- base --#
# (可选) 设置 zsh 为默认 shell
set -g default-shell /bin/zsh

#-- settings --#
set -g mouse on # 开启鼠标切换窗格，按住 shift 复制粘贴
set -g base-index 1 # 窗口编号从 1 开始计数
set -g renumber-windows on # 关掉某个窗口后，编号重排
set -g pane-base-index 1 # 窗格编号从 1 开始计数
set -g display-panes-time 5000 # PREFIX-Q 显示编号的驻留时长，单位 ms
setw -g mode-keys vi # 进入复制模式的时候使用 vi 键位（默认是 EMACS）
setw -g allow-rename off # 禁止活动进程修改窗口名
setw -g automatic-rename off # 禁止自动命名新窗口
set -g default-terminal "tmux-256color" # 开启 256 colors 支持

#-- bindkeys --#
# 以下 3 行设置 ctrl+x 代替 ctrl+b 的快捷键
set -g prefix C-x
unbind C-b
bind C-x send-prefix

# 设置 tmux-prefix + \ 垂直分割窗格
unbind %
bind \ split-window -h
# 设置 tmux-prefix + - 水平分割窗格
unbind '"'
bind - split-window -v

# 设置 ctrl+vim 方式切换窗格
bind -n C-h select-pane -L
bind -n C-j select-pane -D
bind -n C-k select-pane -U
bind -n C-l select-pane -R

# plugins
# tmux plugin manager 插件管理
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
# 保存布局插件，tmux-prefix + ctrl+s/tmux-prefix + ctrl+r 保存 / 恢复
set -g @plugin 'tmux-plugins/tmux-resurrect'
# 自动保存插件
set -g @plugin 'tmux-plugins/tmux-continuum'

# tmux-resurrect 配置
# 恢复 shell 的历史记录, 只有无前台任务运行的窗格 才能被保存
set -g @resurrect-save-bash-history 'on'
# 恢复窗格内容, 目前使用该功能时，请确保 tmux 的 default-command 没有包含 && 或者 || 操作符，
# 否则将导致 bug。（查看 default-command 的值，请使用命令 tmux show -g default-command。）
set -g @resurrect-capture-pane-contents 'on'
# 恢复 vim 会话
set -g @resurrect-strategy-vim 'session'

# set -g @resurrect-save 'S'
# set -g @resurrect-restore 'R'

# tmux-continuum 配置
# 开启自动恢复
set -g @continuum-restore 'on'
# 设置备份间隔（分钟，0 为不自动备份）
set -g @continuum-save-interval '240'
# 状态栏查看备份状态
# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
set -g status-right 'Continuum status: #{continuum_status}'

run '/etc/.tmux/plugins/tpm/tpm'

```

具体配置步骤如下：

1. 编辑 `.tmux.conf` 文件放到你的根目录下；
2. 使用 `git clone https://github.com/tmux-plugins/tpm /etc/.tmux/plugins/tpm` 将 tpm 安装到 `etc` 目录下
(或者随你喜欢，我是个人和 root 共用一套配置，所以放个公共的地)；
3. 输入 `tmux source-file ~/.tmux.conf` 载入配置；
4. 进入 tmux，输入 `※ + U` 查看 tpm 插件更新，弹出页面默认打开命令模式，直接输入 `all` 完成更新；

## 常用控制

{{% callout note %}}
注意，下列所有快捷键区分大小写。
{{% /callout %}}

### 会话

| 按键  | 说明                     |
| ----- | ------------------------ |
| ※ + d | 休眠                     |
| ※ + s | 以菜单方式显示和选择会话 |
| ※ + L | 切换回上一次的会话       |

### 窗口

| 按键    | 说明                                                                             |
| ------- | -------------------------------------------------------------------------------- |
| ※ + c   | 创建新窗口                                                                       |
| ※ + n   | 选择下一个窗口                                                                   |
| ※ + p   | 选择前一个窗口                                                                   |
| ※ + l   | 最近一次活跃窗口之间进行切换                                                     |
| ※ + 0~9 | 选择几号窗口                                                                     |
| ※ + ,   | 重命名窗口                                                                       |
| ※ + .   | 更改窗口的编号，但只能更改成未使用的编号，所以要交换窗口的话，得多次更改进行交换 |
| ※ + &   | 关闭窗口                                                                         |
| ※ + w   | 以菜单方式显示及选择窗口                                                         |
| ※ + f   | 在所有窗口中查找内容                                                             |

### 窗格

| 按键         | 说明                                   |
| ------------ | -------------------------------------- |
| ※ + z        | 最大化 / 还原当前窗格                  |
| ※ + "        | 模向分隔窗格，**替换为了 `-`**         |
| ※ + %        | 纵向分隔窗格，**替换为了 `\`**         |
| ※ + o        | 跳到下一个分隔窗格                     |
| ※ + x        | 关闭窗格                               |
| ※ + ;        | 切换到最后一个使用的窗格               |
| ※ + ↑/↓/←/→  | 切换到上 / 下 / 左 / 右的窗格          |
| ※※ + h/j/k/l | **自定义配置，vim 方式切换窗格**       |
| ※ + q        | 显示窗格编号，并在右上角显示窗格的长宽 |
| ※ + 空格键   | 自动排布窗格，可多次执行尝试多种布局   |

### tpm 插件

| 按键  | 说明                           |
| ----- | ------------------------------ |
| ※ + S | **自定义配置，保存当前布局**   |
| ※ + R | **自定义配置，还原保存的布局** |

## 鼠标操作

鼠标按住窗格的分割线可以修改窗格大小；

如果你用 `wsltty` 或者其他软件，发现右键 / 中键失效，记得按住修饰键 (比如 `Shift`) 再试。
