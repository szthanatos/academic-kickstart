---
title: Ubuntu 易用性配置
linktitle: 配置 Ubuntu
toc: true
type: book
date: 2020-08-22T16:41:03+08:00
lastmod: 2020-08-22T16:41:03+08:00
draft: false
weight: 30
---

## 启用 root 用户

ubuntu 默认没有 root 密码，使用

```bash
sudo passwd root
```

设置 root 密码

## 配置国内镜像源

更换 `/etc/apt/sources.list` 为:

```properties
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

以使用 [清华大学镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

注意我这里 Ubuntu 版本是 20.04(focal)。

关于 Ubuntu 源：

- 按软件自由度区分：

  | 源         | 说明                                     |
  | ---------- | ---------------------------------------- |
  | main       | 完全的自由软件                           |
  | restricted | 不完全的自由软件                         |
  | universe   | ubuntu 官方不提供支持与补丁，全靠社区支持 |
  | muitiverse | 非自由软件，完全不提供支持和补丁         |

- 按使用目的区分：

  | 源        | 说明                                                             |
  | --------- | ---------------------------------------------------------------- |
  | security  | 仅修复漏洞，并且尽可能少的改变软件包的行为，低风险               |
  | update    | 修复严重但不影响系统安全运行的漏洞，低风险                       |
  | proposed  | pre-update，仅建议提供测试和反馈的人进行安装                     |
  | backports | 不会由 Ubuntu-security-team 审查和更新 (但是可能对你有用)，存在风险 |

## 添加第三方源 (可选)

```bash
# git
sudo add-apt-repository ppa:git-core/ppa

# python
sudo add-apt-repository ppa:deadsnakes/ppa

# 更新
sudo apt update
```

## 配置 ssh

修改 `/etc/ssh/sshd_config` 文件：

```properties
# port 为大于 1024 的任意端口（不能是 22 是因为 WSL 和 win10 共用端口且 win10 的优先级更高，win10 内置 SSH Server For Windows 已经占用 22 端口, 大于 1024 是为了避开其他系统服务）
Port 2333

# 允许使用用户名密码方式访问
PasswordAuthentication yes

# (可选) 允许 root 访问 (任何认证方式 / 只允许 public key 认证方式 / 不允许任何认证方式)
PermitRootLogin yes/without-password/no
```

保存，执行以下命令：

```bash
# 生成密钥
dpkg-reconfigure openssh-server

# 重启 ssh
sudo service ssh restart
```

完成 ssh 配置。

早期版本需要重装 `openssh-server`，但是最近直接使用没发现问题。

## 开机自启后台服务 (可选)

在 Wsl 早期，想要使用 Wsl 上的 Python 解释器或者别的工具需要通过 `ssh` 从宿主机连接，而 Wsl 没有 `Systemd` 也没法开机自启。

通过计划任务实现开机启动 Wsl 上的 ssh 服务是一种刚需。

不过现在 Docker 也好，Pycharm 也好，都直接从软件层面支持 Wsl 了，需要 ssh 的场合不多（雾）。

不过 * 2，网上存在的教程都告诉你要在 Ubuntu 写个 shell 脚本，再在 Windows 下写个 vb 脚本，再去创建一个计划任务....

直接计划任务一句话搞定不香么？黑人问号. jpg

还是以 ssh 服务为例：

1. root 身份编辑 `/etc/sudoers` 文件，

   在 `%sudo   ALL=(ALL:ALL) ALL` 下新增一行

   `%sudo   ALL=(ALL) NOPASSWD: /usr/sbin/service ssh *`

   避免 sudo 启动 ssh 需要密码的问题。

2. 在 windows 搜索栏搜索 `计划任务`，或者快速运行窗口输入 `taskschd.msc`，打开任务计划程序

   ![计划任务](https://raw.githubusercontent.com/szthanatos/image-host/master/task-scheduler.png)

3. 创建任务，勾选 `隐藏`，配置改为 win10

   触发器选 `计算机启动时` 或者 `当前用户登录时`，可以配置延迟启动;

   操作是 `启动程序`，程序或脚本选 `C:\Windows\System32\wsl.exe`，添加参数写 `-d Ubuntu -u {Ubuntu 用户名} sudo service ssh start`；

   (可选) 条件中取消勾选 `只有计算机在使用交流电时才启动此任务`；

   (可选) 设置中勾选 `允许按需允许任务`；

   完成

其他需要后台启动的服务类似配置。
