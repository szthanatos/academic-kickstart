---
title: 使用 Docker 开发环境
linktitle: 配置 Docker 环境
toc: true
type: book
date: 2020-08-22T18:07:41+08:00
lastmod: 2020-08-22T18:07:41+08:00
draft: false
weight: 70
---

嗯，是的，其实可以直接在 Windows 下运行 Docker 而无需 Wsl。但是 Wsl2 会提供更好的性能——

如果开启 Wsl2，Docker 会默认把自己的文件系统迁移到 wsl 上（因为在 Win 下的本质也是开了个 Hyper-V 的虚拟机在运行）

——另一方面，咱们前面配好的终端环境也比 Windows 的终端好使...

看到说 Powershell 好的人有，但是真拿它写东西的人好像没见到过...

再一个是 Docker Desktop 可以访问到 Windows 下的文件，而如果直接在 wsl2 中安装 Docker 就没有这份福利了...

所以 Docker Desktop + Wsl2 是最合适的一个组合。

## 安装

1. 下载安装 [Docker Desktop](https://download.docker.com/win/edge/Docker%20Desktop%20Installer.exe) edge 版本
2. 打开设置，检查 `General` 中 `use the WSL2 base egine` 应该是默认勾选的状态；
3. 在 `Resources`-`WSL INTEGRATION` 中检查 `Enable integration with my default WSL Distro` 应该是勾选状态，并将 Linux 发行版置为 `Enable` 状态;
4. 点击 `Apply & Restart`，完成~

Docker Desktop 重启之后，在任意终端 (Windows & Linux) 都可以执行 Docker 命令啦~

## 配置连接 Docker

在 `设置`-`构建、执行、部署` 下找到 `Docker` 配置，可以直接连接 Docker for window：

![连接 docker](https://i.loli.net/2021/06/17/INkgm39Qu1EnjTl.png)

## 编写环境镜像

### 最简运行环境

最简运行环境 Dockerfile 如下：

```dockerfile
FROM python:3.8-buster
LABEL author="Lex Wayne"

WORKDIR /app
COPY . /app
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple &&\
    pip install -U pip &&\
    pip install --no-warn-script-location -r /requirements.txt
```

但是这种比较适用于稳定项目，因为每次更改都会需要重新生成镜像。

代码写到一半，新 `import` 一个第三方包，还得先编辑 `requirements.txt` ，重新生成镜像，才能有提示...画面未免太美。

### 个人开发环境

我惯用的开发环境 Dockerfile 如下：

```dockerfile
FROM python:3.8-buster
LABEL   version="1.1" \
        authors="Lex Wayne" \

ARG DEBIAN_FRONTEND=noninteractive

RUN echo >/etc/apt/sources.list "\n\
deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free\n\
deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free\n\
deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free\n\
deb http://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free\n\
    " &&\
    apt-get update &&\
    apt-get upgrade -y &&\
    apt-get install -y sudo vim openssh-server openssh-client &&\
    mkdir -p /var/run/sshd &&\
    echo 'root:123456' | chpasswd &&\
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config &&\
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd &&\
    echo "export VISIBLE=now" >> /etc/profile

RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
```

使用时相当于后台长期运行一个 Linux 容器环境，通过 ssh 进行远程调试，默认密码 `123456` 请修改掉。

## 运行 Docker 容器

完成 Docker 配置后会在左下方底栏新增一个 `Services`，没有的话从顶部 `视图`-`工具窗口` 中也能找到，

双击连接 Docker，右键运行环境容器；

点击侧边绿色三个箭头的图标即可使用 Dockerfile 部署。

![dockerfile 部署](https://i.loli.net/2021/06/17/zf5aFIpurCqyAQ6.png)

## 参考

- Docker 官方文档: [Docker Desktop WSL 2 backend](https://docs.docker.com/docker-for-windows/wsl/)
- Vs Code 团队博客: [Using Docker in WSL 2](https://code.visualstudio.com/blogs/2020/03/02/docker-in-wsl2)
