---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "RancherOS初步使用小结"
subtitle: ""
summary: ""
authors: []
tags: ['os', 'rancher']
categories: []
date: 2019-09-01T15:39:50+08:00
lastmod: 2019-09-01T15:39:50+08:00
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

{{% toc %}}

## 介绍

`RancherOS`是Rancher推出的一个轻量级的Linux内核操作系统，专为容器环境而设计。

按官网的说法，它具有如下特性：

| 官方说法                             | 瞎翻译                                                                                                    |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| Minimalist OS                        | 极简系统 ——当前版本(v1.5.4)镜像也只有146M，还内置了各类虚拟机工具和N个版本的Docker环境                    |
| Comprehensive System Services        | 综合系统服务——所有的系统服务都可以通过Compose文件声明和启动                                               |
| Improved Security                    | 更安全——没有额外的工具/代码，所有应用都跑在容器里，当然更安全                                             |
| Up-to-Date Version of Docker & Linux | 集成最新的Docker&Linux发行版——装完系统直接就有Docker用，还是最新的，美滋滋                                |
| Automated OS Configuration           | 自动化系统配置——使用cloud-init工具解析`cloud-config`文件，统一管理系统级的所有配置，比如网络，docker源... |
| 24x7 Enterprise-level Support        | 不解释...                                                                                                 |

简单的来说，当我们开始使用容器化的方式来管理应用和服务的时候，我们自然而然的会发现，我们对操作系统其实没什么需求了
——环境依赖和工具链由镜像提供，GUI界面或者浏览器毫无作用，系统内置的各种工具也就是启动一个容器的事...

把这些七七八八的都去掉，最后剩下：一个Linux内核 + Docker环境 + 精简但是统一的配置管理 = RancherOS

![RancherOS 架构](/img/rancheroshowitworks.png)

RancherOS的架构也非常简单，除了内核，就是两个Docker。

一个系统级的Docker(`system-docker`)接管了系统的绝大部分功能，比如在一般Linux上你会用`systemctl restart`重启服务，在这里就是用`system-docker restart`重启一个容器了。

用户级别的Docker也作为一个服务运行在`system-docker`之上，也就是我们一般意义上跑应用的docker，正常使用。

## 安装

1. 在[RancherOS Gitlab页面](https://github.com/rancher/os/releases/)下载对应版本镜像。
2. 新建虚拟机，加载ISO镜像，默认会以`rancher`用户身份进入一个运行在内存之上的临时RancherOS，所以建虚拟机的时候内存可以适当大一点，比如2G
3. 新建/上传一个`cloud-config.yml`文件，主要内容就是写入你的ssh公钥，因为RancherOS安装之后就只能通过ssh+公钥的方式登陆(是的，你在虚拟机控制台都进不去)，一个最简单的示例如下：

    ```yaml
    # cloud-config
    ssh_authorized_keys:
    - ssh-rsa AAA...
    ```
  
4. 没有其他配置了的话，
  
    `sudo ros config valicate -i cloud-config.yml`校验没有格式错误，

    `sudo ros install -c cloud-config.yml -d /dev/sda`将RancherOS安装到硬盘。

    一路只有2个选项，是否要安装？Y，是否要重启？N，因为重启比你卸载光驱还快...直接就又进入一个新的临时RancherOS了...

5. 手动`sudo poweroff`，卸载光驱，重启，看到熟悉的牛头LOGO，恭喜完成~

{{% alert note %}}
如果一定要为rancher设置一个密码的话，将安装命令替换为：

`sudo ros install -c cloud-config.yml -d /dev/sda --append=rancher.password=密码`
{{% /alert %}}

{{% alert note %}}
我遇到的一个情况，在Vmware上安装时，临时RancherOS默认没有网，
需要手动配置网卡：

修改`/etc/network/interfaces`文件，在末尾添加：

```bash
auto eth0
iface eth0 inet static
# IP
address             xxx.xxx.xxx.xxx
# 子网掩码
netmask             xxx.xxx.xxx.xxx
# 广播地址(可选)
broadcast           xxx.xxx.xxx.xxx
# 所在网段(可选)
network             xxx.xxx.xxx.xxx
# 网关
gateway             xxx.xxx.xxx.xxx
# dns服务器
dns-nameservers     xxx.xxx.xxx.xxx
```

配置好执行`sudo ifup eth0`即可连上网络。
{{% /alert %}}

## 使用

整个RancherOS自底向上分三个层面进行管理

| 层面     | 工具          | 管理             |
| -------- | ------------- | ---------------- |
| 系统管理 | ros           | cloud-config.yml |
| 服务管理 | system-docker | 类Compose文件    |
| 应用管理 | docker        | 直接run          |

### ros

ros是对系统进行管理的工具，所以必须要以root权限执行，具体用法可以help看一下：

```bash
$ sudo ros help
NAME:
   ros - Control and configure RancherOS
built: '2019-08-22T07:44:10Z'

USAGE:
   ros [global options] command [command options] [arguments...]

VERSION:
   v1.5.4

AUTHOR(S):
   Rancher Labs, Inc.

COMMANDS:
     config, c   configure settings                               # 配置管理
     console     manage which console container is used           # 切换命令行
     engine      manage which Docker engine is used               # 切换Docker版本
     service, s                                                   # 系统服务管理
     os          operating system upgrade/downgrade               # 内核管理
     tls         setup tls configuration                          # tls管理
     install     install RancherOS to disk                        # 安装系统
     help, h     Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version

```

#### `ros config`

```bash
$ ros config
NAME:
   ros config - configure settings

USAGE:
   ros config command [arguments...]

COMMANDS:
     get       get value                                            # 获取配置
     set       set a value                                          # 设定配置
     images    List Docker images for a configuration from a file   # 没用过
     generate  Generate a configuration file from a template        # 没用过
     export    export configuration                                 # 输出配置，比直接看cloud-config.yml全面
     merge     merge configuration from stdin                       # 合并配置文件
     syslinux  edit Syslinux boot global.cfg                        # 没用过
     validate  validate configuration from stdin                    # 校验配置文件格式
```

所有面向系统的设置都是通过`ros config`进行管理的，默认的配置存放在`/var/lib/rancher/conf/cloud-config.yml`中。未记录的配置修改都会在重启后失效(是的，`sudo passwd`也不能让你下次直接用户名密码登陆，不过有别的办法)

简单的修改配置可以直接执行`ros config set <key> <value>`，比如

`ros config set rancher.docker.tls true`；

多个值写成列表，比如

`ros config set rancher.network.dns.nameservers "['8.8.8.8','8.8.4.4']"`；

复杂一点的可以写一个小的yml，然后执行`ros config merge -i <文件>`进行合并，
理论上也可以手动添加到`/var/lib/rancher/conf/cloud-config.yml`中。
但是个人不推荐这样做，因为现在版本的cloud-config.yml和`ros config export`输出的不是一回事，待研究。

#### `ros service`

```bash
$  ros service
NAME:
   ros service -

USAGE:
   ros service command [command options] [arguments...]

COMMANDS:
     enable   turn on an service              # 启用服务
     disable  turn off an service             # 禁止服务
     list     list services and state         # 服务列表
     delete   delete a service                # 删除服务
     build    Build or rebuild services       # 构建/重构服务
     create   Create services                 # 创建服务
     up       Create and start containers     # 创建并启动服务
     start    Start services                  # 启动服务
     logs     View output from containers     # 查看服务日志
     restart  Restart services                # 重启服务
     stop     Stop services                   # 停止服务
     rm       Delete services                 # 删除服务及镜像
     pull     Pulls service images            # pull服务镜像
     kill     Kill containers                 # 杀死服务容器
     ps       List containers                 # 列出服务容器

OPTIONS:
   --tls               Use TLS; implied by --tlsverify
   --tlsverify         Use TLS and verify the remote [$DOCKER_TLS_VERIFY]
   --tlscacert value   Trust certs signed only by this CA
   --tlscert value     Path to TLS certificate file
   --tlskey value      Path to TLS key file
   --configdir value   Path to docker config dir, default ${HOME}/.docker
   --verbose, --debug
   --help, -h          show help
```

控制的是随系统启动的服务，用法包括声明文件的写法基本和Docker Compose一样，把它理解成Docker Compose的替代品就对了。

别的命令不解释了...

### 系统服务

```bash
$ ros s list
disabled amazon-ecs-agent
disabled container-cron
disabled open-iscsi
disabled zfs
disabled kernel-extras
disabled kernel-headers
disabled kernel-headers-system-docker
enabled  open-vm-tools
disabled hyperv-vm-tools
disabled qemu-guest-agent
disabled rancher-server
disabled rancher-server-stable
disabled amazon-metadata
disabled volume-cifs
disabled volume-efs
disabled volume-nfs
disabled modem-manager
disabled waagent
disabled virtualbox-tools
disabled pingan-amc
```

上面说到系统级的服务都是用`ros s`控制启停，而想要自定义一个系统级的服务的话:

1. 用Docker Compose语法编写服务的`xxx.yml`文件，一般存放到`/var/lib/rancher/conf/`
2. `ros service enable /var/lib/rancher/conf/xxx.yml` 启用该服务
3. `ros service up <serviceName>`启动服务，如果一个Compose里定义了多个服务，那么需要
   `ros service up <serviceName1> <serviceName2> <serviceName3> ...`来同时启动

### docker

没啥好说的...Just use it.

## cloud-config

额外说一下这个cloud-config。

现有的公有云/虚拟化厂商大多支持`cloud-init`工具进行系统配置初始化(某种意义上的事实标准)。cloud-config就是为`cloud-init`服务的。RancherOS在`system-docker`中运行了一个`cloud-init`容器，它会在启动时查找可能位置上的cloud-config文件并依此配置系统配置项。

cloud-config的语法格式就是标准的YAML语法，一个我在用的、比较完整的cloud-config的示例如下：

{{% alert note %}}
使用时请删除掉中文注释...给别人演示的时候懒得删注释结果validate没问题，但是配置就是不生效...
{{% /alert %}}

```yaml
# 主机名
hostname: ros-test

# 系统配置
rancher:
  # 替换控制台为alpine，也可以是ubuntu/centos/debian...
  console: alpine

  # 初始Docker源
  bootstrap_docker:
    registry_mirror: "https://docker.mirrors.ustc.edu.cn/"
  # 系统Docker源
  system_docker:
    registry_mirror: "https://docker.mirrors.ustc.edu.cn/"
  # 用户Docker源
  docker:
    registry_mirror: "https://docker.mirrors.ustc.edu.cn/"

  # 网络
  network:
    interfaces:
      eth0:
        # IP要是CIDR格式，要是和子网掩码对不上就上不了网
        address: 192.168.0.1/24
        # netmask: 255.255.255.0
        # broadcast: 192.168.0.255
        gateway: 192.168.0.254
        mtu: 1500
        dhcp: false
    dns:
      nameservers:
        - 114.114.114.114
        - 8.8.8.8

  # # 扩容现有磁盘不要用fdisk，除非你想把系统格式化了，用这个就能调整磁盘大小
  # resize_device: /dev/sda

# 可登录的机器公钥
ssh_authorized_keys:
  - ssh-rsa ...
  - ssh-rsa ...

# # 挂载新磁盘
# mounts:
# - ["/dev/vdb", "/mnt/s", "ext4", ""]

# 写文件
write_files:
  # 修改apk使用国内镜像
  - path: /etc/apk/repositories
    permissions: "0755"
    owner: root
    content: |
      https://mirrors.ustc.edu.cn/alpine/latest-stable/main
      https://mirrors.ustc.edu.cn/alpine/latest-stable/community

  # 设置CST时区
  - path: /etc/profile
    permissions: "0755"
    owner: root
    content: |
      export CHARSET=UTF-8
      export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
      export PAGER=less
      # 显示样式
      export PS1="\[\e[37m\][\[\e[32m\]\u\[\e[37m\]@\h \[\e[36m\]\w\[\e[0m\]]\\$ "
      # 时区
      export TZ='CST-8'
      umask 022

      for script in /etc/profile.d/*.sh ; do
              if [ -r $script ] ; then
                      . $script
              fi
      done

  # 确保ssh连接时会读取.bashrc
  - path: /home/rancher/.bash_profile
    permissions: "0755"
    owner: rancher
    content: |
      # If the shell is interactive and .bashrc exists, get the aliases and functions
      if [[ $- == *i* && -f ~/.bashrc ]]; then
          . ~/.bashrc
      fi

  # 配置.bashrc
  - path: /home/rancher/.bashrc
    permissions: "0755"
    owner: rancher
    content: |
      # .bashrc
      # User specific aliases and functions
      alias  d="docker "
      alias di="docker image"
      alias dc="docker container"
      alias dv="docker volumn"
      alias dn="docker netwrok"

      # Source global definitions
      if [ -f /etc/bashrc ]; then
              . /etc/bashrc
      fi

# 启动时执行命令
runcmd:
  #   # 两种写法
  #   - [touch, /home/rancher/test1]
  #   - echo "test" > /home/rancher/test2
  # 开机更新apk源
  - apk update
  # 启动定时任务服务
  - crond

```
