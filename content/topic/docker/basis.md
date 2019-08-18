---
title: Docker -01- 基本概念
linktitle: 基本概念
toc: true
type: docs
date: "2018-12-05T22:17:22+08:00"
draft: false

menu:
  docker:
    # parent: 基本概念
    weight: 1

weight: 1
---

## Docker简介

### 什么是Docker

Docker 最初是 dotCloud 公司创始人 Solomon Hykes 在法国期间发起的一个公司内部项目，并于2013 年 3 月以 Apache 2.0 授权协议开源，主要项目代码在 [GitHub](https://github.com/moby/moby) 上进行维护，后来还加入了 Linux 基金会，并成立推动 [开放容器联盟（OCI）](https://www.opencontainers.org/)。

Docker 最初是在 Ubuntu 12.04 上以[Go 语言](https://golang.org/) 进行开发实现的, Red Hat 则从 RHEL 6.5 开始对 Docker 进行支持(换句话说不支持CentOS6.5以下)。

Docker 是一种 **容器化技术** ，类似虚拟机的概念，但不同的是传统虚拟机是在虚拟硬件的基础上，完整模拟一整个操作系统，而Docker是以单个应用（容器）为单位进行虚拟。

![传统虚拟化](/img/virtualization.png)

![Docker](/img/docker.png)

### Docker特点

Docker具有以下特点：

- **文件系统隔离** ：每个进程容器运行在完全独立的根文件系统里。
- **资源隔离** ：可以使用cgroup为每个进程容器分配不同的系统资源，例如CPU和内存。
- **网络隔离** ：每个进程容器运行在自己的网络命名空间里，拥有自己的虚拟接口和IP地址。
- **写时复制** ：采用写时复制方式创建根文件系统，这让部署变得极其快捷，并且节省内存和硬盘空间。
- **日志记录** ：Docker将会收集和记录每个进程容器的标准流（stdout/stderr/stdin），用于实时检索或批量检索。
- **变更管理** ：容器文件系统的变更可以提交到新的映像中，并可重复使用以创建更多的容器。无需使用模板或手动配置。
- **交互式Shell** ：Docker可以分配一个虚拟终端并关联到任何容器的标准输入上，例如运行一个一次性交互shell。

### 为什么要使用Docker

| 特性       | 容器               | 虚拟机      |
| :--------- | :----------------- | :---------- |
| 启动       | 秒级               | 分钟级      |
| 硬盘使用   | 一般为 `MB`        | 一般为 `GB` |
| 性能       | 接近原生           | 弱于原生    |
| 系统支持量 | 单机支持上千个容器 | 一般几十个  |

- **更高效的利用系统资源** ：由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，Docker 对系统资源的利用率更高。相比虚拟机技术，一个相同配置的主机，往往可以运行更多数量的应用。
- **更快速的启动时间** ：Docker 容器应用，由于直接运行于宿主内核，无需启动完整的操作系统，因此可以做到秒级、甚至毫秒级的启动时间。
- **一致的运行环境** ： Docker 的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性，从而不会再出现 _「这段代码在我机器上没问题啊」_ 这类问题。
- **持续交付和部署** ：对`DevOps`人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员可以通过 [Dockerfile](###Dockerfile) 来进行镜像构建，并结合 `持续集成(Continuous Integration)` 系统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合 `持续部署(Continuous Delivery/Deployment)`系统进行自动部署。
- **更轻松的迁移** ：由于 Docker 确保了执行环境的一致性，使得应用的迁移更加容易。Docker 可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运行结果是一致的。因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。

## 基本概念

Docker 包括三个基本概念

- 镜像（`Image`）
- 容器（`Container`）
- 仓库（`Repository`）

理解了这三个概念，就理解了 Docker 的整个生命周期。

### 镜像

操作系统分为内核和用户空间。对于 Linux 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 `root` 文件系统。比如官方镜像 `ubuntu:16.04` 就包含了完整的一套 Ubuntu 16.04 最小系统的 `root` 文件系统。

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。 **镜像不包含任何动态数据，其内容在构建之后也不会被改变。**

严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

**镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。** 比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

### 容器

镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

**容器的实质是进程** ，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 `命名空间`。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。

前面讲过镜像使用的是分层存储，容器也是如此。 **每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，** 我们可以称这个为容器运行时读写而准备的存储层为 **容器存储层** 。

容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此， **任何保存于容器存储层的信息都会随容器删除而丢失** 。

按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。 **所有的文件写入操作，都应该使用 [数据卷（Volume）](####方式1：数据卷（推荐）)、或者[绑定宿主目录](####方式2：挂载主机目录)** ，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

### 仓库

如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker提供注册服务器(`Docker Registry`)来实现这样的服务。

一个`Docker Registry`中可以包含多个 **仓库** （`Repository`）；每个仓库可以包含多个 **标签** （`Tag`）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。

以 [Ubuntu 镜像](https://store.docker.com/images/ubuntu) 为例，`ubuntu` 是仓库的名字，其内包含有不同的版本标签，如，`14.04`, `16.04`。我们可以通过 `ubuntu:14.04`，或者 `ubuntu:16.04` 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 `ubuntu`，那将视为 `ubuntu:latest`。

仓库名经常以 *两段式路径* 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

类似git 和GitHub，官方提供[Docker Hub](https://hub.docker.com/)，作为默认的 Registry。用户也可以在本地搭建私有 Docker Registry。Docker 官方提供了 [Docker Registry](https://store.docker.com/images/registry/) 镜像，可以直接使用做为私有 Registry 服务。

### 生命周期

结合上面的概念，这里有一张图比较好的概括了整个Docker工作的生命周期（以及主要命令）。
![生命周期](/img/period.png)

## 安装配置

这里仅以CentOS 安装 Docker CE 举例说明。详见[Docker 官方 CentOS 安装文档](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)

### 准备工作

#### 系统要求

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 `overlay2` 存储层驱动）无法使用，并且部分功能可能不太稳定。

> 警告：切勿在没有配置 Docker YUM 源的情况下直接使用 yum 命令安装 Docker.

#### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```bash
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-selinux \
                docker-engine-selinux \
                docker-engine
```

### 使用脚本安装（非生产环境）

对于个人测试，可以使用这个脚本自动化安装Docker：

```bash
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```

但是，需要注意， **这个脚本可能扰乱你的系统配置、安装及大量的（你可能用不到的）依赖，并且只能安装最新（可能未经充分测试的）版本的Docker** ， 所以不推荐在生产环境中使用。

### 使用 yum 安装

安装依赖包：

```bash
sudo yum install -y yum-utils \
                    device-mapper-persistent-data \
                    lvm2
```

添加 `yum` 软件源：

```bash
# 中国科学技术大学开源软件镜像源
sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo


# 官方源
# sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo
```

更新 `yum` 软件源缓存，并安装 `docker-ce`。

```bash
sudo yum makecache fast
sudo yum install docker-ce
```

### 离线安装

以docker-ce-18.03.1为例：

1. 在`https://download.docker.com/linux/centos/7/x86_64/stable/Packages/`这里找到对应rpm包
2. 执行安装命令：`rpm -ivh docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm`
3. 由于安装环境不同，可能会发现缺少一些相关依赖包（eg: libcgroup、libtool-ltdl、container-selinux）前往`https://pkgs.org/`或`https://buildlogs.centos.org/`下载对应依赖包，依次安装即可

### 启动 Docker CE

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### 建立 Docker 用户组

默认情况下，docker命令需要`root`权限，为了避免每次输入命令都要加`sudo`，可以将用户加入 `docker` 用户组：

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

### 测试 Docker 是否安装正确

执行

```bash
docker run hello-world
```

Docker会从官方仓库下载hello-world镜像并启动，如果一切正常的话会看到类似如下提示：

```bash
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:be0cd392e45be79ffeffa6b05338b98ebb16c87b255f48e297ec7f98e123905c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

### 镜像加速

鉴于国内网络问题，建议使用Docker中国或者其他国内镜像源。

修改（或新增）`/etc/docker/daemon.json`文件，添加:

```bash
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

之后重启Docker使配置生效。

### 常用Docker操作

```bash
# 查看docker版本
docker version
# 显示docker系统的信息
docker info
# 日志信息
docker logs
# 故障检查
service docker status
# 启动关闭docker
sudo service docker start|stop
```

## 使用镜像

### 基本操作

以redis为例，我们从[Docker Hub](https://hub.docker.com/explore/)上获取官方镜像到本地：

![Docker hub redis](/img/hub-redis.jpg)

```bash
docker pull redis
```

_ps1：_ 由于redis是官方源（Official），否则应该写完整的两段式仓库名 `<用户名>/<软件名>`，例如bitnami/redis。

_ps2：_ 此处没有指定镜像版本，默认会拉取redis:lastest镜像，指定版本应该写成例如：redis:5.0-rc5

查看已经下载的镜像：

```bash
docker image ls

# 会有类似如下显示
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
redis                latest              5f515359c7f8        5 days ago          183 MB
......
```

更细节的显示可以使用`docker image ls --format "{{.ID}}: {{.Repository}}"`直接列出镜像ID和仓库名,

或者使用`docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"`
以表格等距显示.

如果要删除某个镜像的话，可以使用`docker image rm {IMAGE ID}|{REPOSITORY}`命令，不要过先确保没有容器在使用这个镜像。

### Dockerfile

除了引用制作好的镜像，我们也可以基于现有镜像定制新的镜像。定制所用的脚本文件就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)** ，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

我们新建一个空白文件，命名为 `dockerfile`，再文件中写入如下内容：

```dockerfile
FROM redis
RUN mkdir redis
WORKDIR redis
COPY ./redis.conf /etc/
CMD ["redis-server", "/etc/redis.conf"]
```

我们依次解释上面每一行：

- **FROM** 就是指定 **基础镜像** ,一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。如果不以任何镜像为基础，那应该用`FROM scratch`作为起始指令。
- **RUN** 是Dockerfile的核心指令，用于执行一条命令，由于Dockerfile 每一条指令都会新建一层，所以应该尽量将执行的内容写在一行（多行内容可以通过在末尾加`\`以表示未结束），它有两种写法：
  - **shell** 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。
  - **exec** 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。
- **WORKDIR** 表示指定当前工作目录，相当于`cd`命令。
- **COPY** 即复制文件到容器中，在这里是把redis.conf文件复制到容器的`/etc`目录下。
- **CMD** 是启动程序的命令，写法和`RUN`相同，一般推荐使用`exec`格式。

常用Docker指令列表如下：

| 指令        | 含义             | 用法                                          |
| ----------- | ---------------- | --------------------------------------------- |
| FROM        | 指定基础镜像     | `FROM <基础镜像>`                             |
| RUN         | 执行指令         | `RUN ["可执行文件", "参数1", "参数2"]`        |
| COPY        | 复制文件         | `COPY ["<源路径1>",... "<目标路径>"]`         |
| ADD         | 更高级的复制文件 | `ADD "<压缩文件>"`                            |
| CMD         | 容器启动命令     | `CMD ["可执行文件", "参数1", "参数2"...]`     |
| ENTRYPOINT  | 入口点           | `ENTRYPOINT ["可执行文件", "参数1", "参数2"]` |
| ENV         | 设置环境变量     | `ENV <key1>=<value1> <key2>=<value2>...`      |
| ARG         | 构建参数         | `ARG <参数名>[=<默认值>]`                     |
| VOLUME      | 定义匿名卷       | `VOLUME ["<路径1>", "<路径2>"...]`            |
| EXPOSE      | 暴露端口         | `EXPOSE <端口1> [<端口2>...]`                 |
| WORKDIR     | 指定工作目录     | `WORKDIR <工作目录路径>`                      |
| USER        | 指定当前用户     | `USER <用户名>`                               |
| HEALTHCHECK | 健康检查         | `HEALTHCHECK NONE | [选项] CMD <命令>`        |
| ONBUILD     | 构建下级镜像     | `ONBUILD <其它指令>`                          |
| MAINTAINER  | 指定作者         | `ONBUILD <作者>`                              |

更多指令及用法请参照[官方文档](https://docs.docker.com/engine/reference/builder)

如上，我们完成了一个使用自己配置文件的redis镜像的准备工作，之后依据这个Dockerfile进行构建：

```bash
docker build -t redis_test:v0.1 .

# 会有类似如下输出：
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM redis
...
...
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```

`docker build`的用法为：

```bash
docker build [选项] <上下文路径/URL/->
```

最后，可以使用`docker push`将你自己构建的镜像上传到仓库中，详细用法见[官方文档 push](https://docs.docker.com/engine/reference/commandline/push/)

## 容器操作

### 容器启停

我们可以用这样的方式从之前的镜像启动一个容器：

```bash
docker run -d --name some-redis redis
```

`docker run`的用法为`docker run [选项] 镜像 [命令] [参数...]`，其中：

`--name` 指定容器的名称， `-d` 指定后台运行，其他常用参数包括`-i` 交互式操作，`-t` 使用终端（`it`一般同时使用），`--rm` 容器退出后随之将其删除，完整参数列表可以通过`--help`或者[在线文档 docker run](https://docs.docker.com/engine/reference/commandline/run/)查看

由于我们是在后台运行，使用`docker container ls`来查看容器相关情况，如果要查看停止的进程，后面需要增加参数`-a`：

```bash
docker container ls

# 会看到类似如下内容
CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
77b2dc01fe0f  redis:v2  redis-server redis.conf 'while tr  2 minutes ago  Up 1 minute        agitated_wright
```

使用`docker container stop`来结束容器的运行：

```bash
docker container stop 77b2dc01fe0f
```

类似的，使用`docker container start | restart | stop`可以控制容器的启停，
使用`docker container rm` 来删除指定容器。

### 数据管理

之前提到过，随着容器的销毁，容器内的数据也会一同丢失。为了保存数据，Docker提供了两种方式（还有一种tmpfs mountsb不常用到）：

#### 方式1：数据卷（推荐）

数据卷 `volume` 是一个可供一个或多个容器使用的特殊目录，它不依赖于Unix文件系统，也拥有独立于容器的生命周期。

创建一个数据卷:

```bash
docker volume create my-vol
```

查看数据卷及具体信息：

```bash
# 查看所有的数据卷
docker volume ls

# 会看到类似如下内容
local               my-vol

# -----------------------------------

# 查看具体卷的信息
docker volume inspect my-vol

# 会看到类似如下内容
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

在用 docker run 的时候，增加 `--mount` 参数来使用数据卷,还是以启动redis为例，这里我们启动redis并且开启aof持久化：

```bash
docker run -d \
    --name redis \
    --mount source=my-vol,target=/data \
    # -v my-vol:/data \
    redis \
    redis-server --appendonly yes
```

在这里redis产生的数据（`/data`目录下）被挂载到数据卷`my-vol`中。

我们也可以使用`-v`或者`--volume`语法，但是[官方建议](https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag)尽量使用`--mount`。

同样使用`inspect`语法，我们可以查看redis容器的信息：

```bash
docker inspect redis

# 会看到类似如下内容
"Mounts": [
    {
        "Type": "volume",
        "Name": "my-vol",
        "Source": "/var/lib/docker/volumes/my-vol/_data",
        "Destination": "/data",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

#### 方式2：绑定主机目录

我们也可以直接将容器的数据挂载 `bind mount`到宿主机的目录或文件 （而非由Docker创建的数据卷）,以当前目录`$(pwd)`为例：

```bash
docker run -d \
    --name redis \
    --mount type=bind,source="$(pwd)"/target,target=/data \
    redis \
    redis-server --appendonly yes
```

挂载单独文件的方法类似。

需要注意，本地目录必须存在，否则会报错。

#### 区别

![types of mounts volume](/img/types-of-mounts-volume.png)

Volumes是由Docker创建和管理，存储在宿主机固定位置（在linux上是/var/lib/docker/volumes/）。 **非Docker应用程序不能改动这一位置的数据。** 一个数据卷可以同时被挂载到几个容器中。即使没有正在运行的容器使用这个数据卷，它依然不会清除。可以通过`docker volume prune`清除不再使用的数据卷。

Bind mounts的数据可以存放在宿主机的任何地方。 **非Docker应用程序可以改变这些数据。**

### 使用网络

#### 端口映射

`docker run`的时候使用`-P`(--publish-all)参数，随机映射一个 49000~49900 的端口到内部容器开放的网络端口。

或者使用`-p ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`(--publish)来指定具体端口映射：

```bash
docker run -d \
    --name some-redis \
    -p 6379:6379 \
    -p 127.0.0.1::16379/udp
    -p 127.0.0.1:80:80
    redis  
```

这里我们分别将容器的6379端口映射到宿主机 **任意ip的6379端口** ，容器的16379 udp端口映射到宿主机的 **任意端口** ，容器的80端口映射到宿主机 **对应的80端口** 。

使用`docker port` 可以查看对应容器的全部端口映射。

#### 容器互联

简单的容器互联可以通过`--link` 实现，但是 **官方未来可能会删除这个参数** ，所以不展开。

最新的方式是搭建docker网络实现容器互联，先创建一个新的 Docker 网络：

```bash
docker network create -d bridge my-net
```

这里的`-d` 参数指定网络类型，常用的只有bridge，其他的可能会在Swarm用到,如果不知道Swarm是什么就不用在意。

以redis客户端/服务端为例，分别在启动的时候将之加入`my-net`网络：

```bash
docker run -d \
    --name redis-server \
    --network my-net \
    redis

docker run -it \
    --rm \
    --name redis-client \
    --network my-net \
    redis redis-cli -h redis-server

```

可以看到成功进入redis-cli客户端，我们可以尝试`info`/`keys *`或者其他命令查看redis服务端运行情况。

## 延申

### 容器编排

面临一组容器配合使用的情况，例如一个包括负载均衡——网站后台——数据库的Web系统，我们可以使用Docker提供的[Compose](https://github.com/docker/compose)完成统一配置管理。它将提供相同功能的容器定义为服务`service`——以方便复用；将完整的容器组合组成项目`project`以方便统一管理。所有的配置通过一个yml文件即可实现。

### Nvidia Docker

对使用GPU的容器，Docker提供[Nvidia Docker](https://github.com/NVIDIA/nvidia-docker)以发挥GPU的运算性能。

基本要求如下：

- GNU/Linux x86_64 with kernel version > 3.10
- Docker >= 1.12
- NVIDIA GPU with Architecture > Fermi (2.1)
- NVIDIA drivers ~= 361.93 (untested on older versions)

详细安装使用见[官方项目Wiki](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0))
