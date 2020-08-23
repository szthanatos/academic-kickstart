---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: 使用tmux复用终端
linktitle: Tmux配置
toc: true
type: docs
date: 2020-08-22T17:53:59+08:00
lastmod: 2020-08-22T17:53:59+08:00
draft: false
menu:
  wsl2:
    weight: 6

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 6
---

Tmux是一个终端复用软件。比如你开一个terminal窗口，相当于和本地的或远程的Linux建立了一次会话连接。

如果你想多做几件事，比如一个窗口运行服务，一个窗口运行客户端，对不起，你得再开一个窗口，再建立一个连接。

tmux就是解决这个问题的工具，它可以让你的终端(Terminal)给复用(mux)了，在一次连接里面做无数多件不同的事情。

Ubuntu自带Tmux，无需额外安装。

## 概念

![tmux](https://raw.githubusercontent.com/szthanatos/image-host/master/tmux.jpg)

Tmux有这么几个概念：

- Session：输入tmux后就创建了一个会话，一个会话是一组窗体的集合。
- Window：会话中一个可见的窗口，你的屏幕一次只会看到一个窗口的内容。
- Pane:一个窗口可以分成多个窗格。

使用tmux的过程可以分为：

- 控制模式（按下或者按住前缀(tmux-prefix)，默认ctrl+b, 下文用※表示），也就是用快捷键执行动作；
- 命令模式（输入tmux 后接命令），执行内部命令，好理解；
- 一般模式，就是把窗格内的东西当一般terminal用；

## 配置

添加配置文件`.tmux.conf`，内容如下：

```properties
#-- base --#
# (可选)设置zsh为默认shell
set -g default-shell /bin/zsh

#-- settings --#
# 开启鼠标切换窗格，按住shift复制粘贴
set -g mouse on
# 窗口编号从 1 开始计数
set -g base-index 1
# 关掉某个窗口后，编号重排
set -g renumber-windows on
# 窗格编号从 1 开始计数
set -g pane-base-index 1
# PREFIX-Q 显示编号的驻留时长，单位 ms
set -g display-panes-time 5000
# 进入复制模式的时候使用 vi 键位（默认是 EMACS）
setw -g mode-keys vi
# 禁止活动进程修改窗口名
# setw -g allow-rename off
# 禁止自动命名新窗口
setw -g automatic-rename off
# 开启256 colors支持
set -g default-terminal "tmux-256color"

#-- bindkeys --#
# 以下3行设置ctrl+x代替ctrl+b的快捷键
set -g prefix C-x
unbind C-b
bind C-x send-prefix

# 设置tmux-prefix + \垂直分割窗格
unbind %
bind \\ split-window -h
# 设置tmux-prefix + -水平分割窗格
unbind '"'
bind - split-window -v

# 设置ctrl+vim方式切换窗格
bind -n C-h select-pane -L
bind -n C-j select-pane -D
bind -n C-k select-pane -U
bind -n C-l select-pane -R
```

有了这个配置文件立刻就能生效。

## 常用操作

下面用`※`代表tmux前缀，也就是`ctrl+x`，默认是`ctrl+b`。

### 鼠标

tmux中执行有些正常terminal中的鼠标操作需要按住`shift`，多试试。

窗格的分割线可以直接用鼠标拖动。

### 命令

#### 会话

- `tmux` 新建无名称会话
- `tmux new -s demo` 新建名称为demo的会话
- `tmux detach` 断开当前会话，既※ + d
- `tmux a` 默认进入第一个会话
- `tmux a -t demo` 进入到名称为demo的会话
- `tmux list-session` 查看所有会话
- `tmux ls` 同上，简写

#### 结束

- `tmux kill-server` 关闭服务器，所有的会话都将关闭
- `tmux kill-session -t demo` 关闭demo会话
- `tmux kill-window` 关闭窗口
- `tmux kill-pane` 关闭窗格

### 控制

#### 会话(Session)

- `※ + d` 休眠
- `※ + s` 以菜单方式显示和选择会话
- `※ + L` 切换回上一次的会话

#### 窗口(Windows)

- `※ + c` 创建新窗口
- `※ + n` 选择下一个窗口
- `※ + p` 选择前一个窗口
- `※ + l` 最近一次活跃窗口之间进行切换
- `※ + 0~9` 选择几号窗口
- `※ + ,` 重命名窗口
- `※ + .` 更改窗口的编号，但只能更改成未使用的编号，所以要交换窗口的话，得更改多次进行交换
- `※ + &` 关闭窗口
- `※ + w` 以菜单方式显示及选择窗口
- `※ + f` 在所有窗口中查找内容

#### 窗格(Pane)

- `※ + z` 最大化/还原当前窗格
- `※ + "` 模向分隔窗格，替换为了-
- `※ + %` 纵向分隔窗格，替换为了\
- `※ + o` 跳到下一个分隔窗格
- `※ + x` 关闭窗格
- `※ + ;` 切换到最后一个使用的窗格
- `※ + 上下键` 上一个及下一个分隔窗格
- `※ + 空格键` 切换窗格布局

## tmux插件

之前还用过一段时间Tmux的插件，后来发现需要长期停留在Tmux的时候很少，所以后面没用了，

配置文件还是放出来留给有缘人：

```properties
# plugins
# tmux plugin manager 插件管理
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
# 保存布局插件，tmux-prefix + ctrl+s/tmux-prefix + ctrl+r保存/恢复
set -g @plugin 'tmux-plugins/tmux-resurrect'
# 自动保存插件
set -g @plugin 'tmux-plugins/tmux-continuum'

# tmux-resurrect配置
# 恢复shell的历史记录,只有无前台任务运行的窗格 才能被保存
set -g @resurrect-save-bash-history 'on'
# 恢复窗格内容,目前使用该功能时，请确保tmux的default-command没有包含&& 或者||操作符，
# 否则将导致bug。（查看default-command的值，请使用命令tmux show -g default-command。）
set -g @resurrect-capture-pane-contents 'on'
# 恢复vim会话
set -g @resurrect-strategy-vim 'session'

# set -g @resurrect-save 'S'
# set -g @resurrect-restore 'R'

# tmux-continuum配置
# 开启自动恢复
set -g @continuum-restore 'on'
# 设置备份间隔（分钟，0为不自动备份）
set -g @continuum-save-interval '240'
# 状态栏查看备份状态
# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
set -g status-right 'Continuum status: #{continuum_status}'

run '/etc/.tmux/plugins/tpm/tpm'
```

tpm通过git安装：

```shell
git clone https://github.com/tmux-plugins/tpm  /etc/.tmux/plugins/tpm
```

安装tpm后需要重新读取配置文件生效：

1. 进入tmux，输入`※ + :`进入命令模式；
2. 再输入`source-file ~/.tmux.conf`手动刷新配置文件；
3. 最后输入`※ + shift + u`进入tpm插件升级页面进行升级。
