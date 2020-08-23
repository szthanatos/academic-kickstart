---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: Ubuntu易用性配置
linktitle: Ubuntu配置
toc: true
type: docs
date: 2020-08-22T16:41:03+08:00
lastmod: 2020-08-22T16:41:03+08:00
draft: false
menu:
  wsl2:
    weight: 4

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

## 启用root用户

ubuntu 默认没有root 密码，使用

```bash
sudo passwd root
```

设置root 密码

## 配置国内镜像源

更换`/etc/apt/sources.list`为:

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

以使用[清华大学镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

注意我这里Ubuntu版本是20.04(focal)。

关于Ubuntu源：

- 按软件自由度区分：

  | 源         | 说明                                     |
  | ---------- | ---------------------------------------- |
  | main       | 完全的自由软件                           |
  | restricted | 不完全的自由软件                         |
  | universe   | ubuntu官方不提供支持与补丁，全靠社区支持 |
  | muitiverse | 非自由软件，完全不提供支持和补丁         |

- 按使用目的区分：

  | 源        | 说明                                                             |
  | --------- | ---------------------------------------------------------------- |
  | security  | 仅修复漏洞，并且尽可能少的改变软件包的行为，低风险               |
  | update    | 修复严重但不影响系统安全运行的漏洞，低风险                       |
  | proposed  | pre-update，仅建议提供测试和反馈的人进行安装                     |
  | backports | 不会由Ubuntu-security-team审查和更新(但是可能对你有用)，存在风险 |

## 添加第三方源(可选)

```bash
# git
sudo add-apt-repository ppa:git-core/ppa

# python
sudo add-apt-repository ppa:deadsnakes/ppa

# 更新
sudo apt update
```

## 配置ssh

修改`/etc/ssh/sshd_config`文件：

```properties
# port为大于1024的任意端口（不能是22是因为WSL和win10共用端口且win10的优先级更高，win10内置SSH Server For Windows已经占用22端口,大于1024是为了避开其他系统服务）
Port 2333

# 允许使用用户名密码方式访问
PasswordAuthentication yes

# (可选)允许root访问(任何认证方式/只允许public key认证方式/不允许任何认证方式)
PermitRootLogin yes/without-password/no
```

保存，执行以下命令：

```bash
# 生成密钥
dpkg-reconfigure openssh-server

# 重启 ssh
sudo service ssh restart
```

完成ssh配置。

早期版本需要重装`openssh-server`，但是最近直接使用没发现问题。

## 开机自启后台服务(可选)

在Wsl 早期，想要使用Wsl 上的Python解释器或者别的工具需要通过`ssh`从宿主机连接，而Wsl 没有`Systemd`也没法开机自启。

通过计划任务实现开机启动Wsl 上的ssh服务是一种刚需。

不过现在Docker也好，Pycharm也好，都直接从软件层面支持Wsl了，需要ssh的场合不多。

不过*2，网上存在的教程都告诉你要在Ubuntu写个shell脚本，再在Windows下写个vb脚本，再去创建一个计划任务....

直接计划任务一句话搞定不香么？黑人问号.jpg

还是以ssh服务为例：

1. root身份编辑`/etc/sudoers`文件，

   在`%sudo   ALL=(ALL:ALL) ALL`下新增一行

   `%sudo   ALL=(ALL) NOPASSWD: /usr/sbin/service ssh *`

   避免sudo 启动ssh需要密码的问题。

2. 在windows搜索栏搜索`计划任务`，或者快速运行窗口输入`taskschd.msc`，打开任务计划程序

   ![计划任务](https://raw.githubusercontent.com/szthanatos/image-host/master/task-scheduler.png)

3. 创建任务，勾选`隐藏`，配置改为win10

   触发器选`计算机启动时`或者`当前用户登录时`，可以配置延迟启动;

   操作是`启动程序`，程序或脚本选`C:\Windows\System32\wsl.exe`，添加参数写`-d Ubuntu -u {Ubuntu用户名} sudo service ssh start`；

   (可选)条件中取消勾选`只有计算机在使用交流电时才启动此任务`；

   (可选)设置中勾选`允许按需允许任务`；

   完成

其他需要后台启动的服务类似配置。
