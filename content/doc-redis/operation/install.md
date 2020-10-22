---
title: "Redis 单机 & 集群安装"
linktitle: "单机 & 集群安装"
toc: true
type: book
date: 2018-08-30T16:40:48+08:00
draft: false
weight: 41
---

## 版本

| 软件                                                               | 版本   | 更新日期   | 备注                                 |
| ------------------------------------------------------------------ | ------ | ---------- | ------------------------------------ |
| [Redis](http://download.redis.io/releases/redis-4.0.11.tar.gz)     | 4.0.11 | 2018-08-03 | 必装                                 |
| [Ruby](https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.1.tar.gz) | 2.5.1  | 2018-03-28 | 可选, 集群执行 rb 脚本的节点安装即可 |
| [rubygem](https://rubygems.org/rubygems/rubygems-2.7.7.tgz)        | 2.7.7  | 2018-05-18 | 同上                                 |
| [gem-redis](https://rubygems.org/downloads/redis-4.0.2.gem)        | 4.0.2  | 2018-08-13 | 同上                                 |

## 安装

1. 以 CentOS 7 为例
2. 从上述链接下载 redis 文件
3. 安装环境依赖 `yum install -y gcc gcc-c++ tcl`
4. `tar -xvzf {redis.tar.gz}` 解压
5. 进入 redis 目录，执行 `make && make install` 进行安装

    _如果 make 出错，通过 make test 检查：_

    - 尝试只用单核运行：`taskset -c 1 sudo make test`
    - 更改 `tests/integration/replication-psync.tcl` 文件, 把对应报错的那段代码中的 `after 100` 改成 `after 500`

6. 完成单节点安装

## 配置

默认配置文件为位于 redis 目录下的 `redis.conf`

### 最小配置

一个最小的 redis.conf 配置如下：

```properties
# 节点监听的端口号
port {your_port}
# 是否以进程守护方式 (后台) 运行
daemonize yes
# 允许访问的 IP 地址，设置为 0.0.0.0 的时候可以从任意 IP 访问 redis，多个 ip 用逗号隔开
bind {your_ip}
# 工作目录，数据存放位置
dir {your_dir}
# 进程文件名称，固定为 redis_监听的端口号. pid
pidfile /var/run/redis_{your_port}.pid
```

完成以上配置即可启动单节点 redis。

### 集群配置

集群需要在 redis.conf 中配置以下部分：

```properties
# 以集群模式启动
cluster-enabled yes
# 集群配置存放的文件名，一般为 node - 端口号. conf
cluster-config-file nodes-{port}.conf
# 集群超时
cluster-node-timeout 15000
# 是否启用 aof 方式持久化，建议开启
appendonly yes
```

## 启动

### 防火墙

redis 需要使用指定端口号以及指定端口号 + 10000 进行通讯，以 6379 端口为例，如果开启了防火墙，需要执行：

```bash
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --zone=public --add-port=16379/tcp --permanent
firewall-cmd --reload
```

### 单结点启动

执行

```bash
redis-server {dir}/redis.conf
```

启动 redis

### 集群启动

#### ruby 环境

redis 集群是通过 Ruby 编写的脚本进行联通的（但不需要在每隔节点都执行），所以集群中起码一个节点，应该具备 ruby 环境、rubygem 包管理软件、rubygem 中的 redis 包。

ruby 环境搭建过程如下：

1. yum 安装 ruby 环境及包

    ```bash
    yum -y install ruby ruby-devel rubygems
    ```

2. 修改 ruby 源，使用国内镜像

    ```bash
    gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
    # 如果修改失败将 https 换为 http 重试
    gem sources --add http://gems.ruby-china.org/ --remove http://rubygems.org/
    ```

3. 检查源列表，确保只有 gems.ruby-china.org

    ```bash
    gem sources -l
    ```

4. 安装 ruby 的 redis 包：

    ```bash
    gem install redis
    ```

#### 联通集群

1. 启动所有节点上的 redis 服务
2. 进入 redis 安装路径下 src 文件夹，执行集群命令：

    ```bash
    # --replicas 1 表示主从复制比例为 1:1，即一个主节点对应一个从节点
    ./redis-trib.rb create --replicas 1 {node1_ip:port} {node2_ip:port} ...
    ```

3. 默认会自动分配主从节点，确认的话输入 `yes` 完成集群的创建

## 检查集群状态

```bash
# -c 表示连接的是集群
redis-cli -c -h {ip} -p {port}
# 查看集群节点
> cluster nodes
# 查看集群信息
> cluster info
```

## 相关命令

除了使用 rb 脚本，其实可以直接在 redis 节点上 [通过命令操作集群](/post/redis/redis_command.md)，个人更推荐这个做法。

主要用到的是

{{% callout note %}}
谨慎使用 rb 脚本对 redis 进行修复，从来没修复成功过...

它还会把你的哈希槽按顺序平均分配到所有节点上，本来可能是 A 节点管理 0-6000，B 节点 6001-12000...fix 完了之后就成了 A 节点：1，3，5，7... 终端都被坑到无法阅读了。
{{% /callout %}}

## 附注

### 离线安装 redis 环境

离线情况需要本地下载如下 rpm 包 (版本号以最新为准):

```bash
cpp-4.8.5-16.el7.x86_64.rpm
gcc-4.8.5-16.el7.x86_64.rpm
gcc-c++-4.8.5-16.el7.x86_64.rpm
glibc-2.17-196.el7.i686.rpm
glibc-2.17-196.el7.x86_64.rpm
glibc-common-2.17-196.el7.x86_64.rpm
glibc-devel-2.17-196.el7.x86_64.rpm
glibc-headers-2.17-196.el7.x86_64.rpm
libgcc-4.8.5-16.el7.i686.rpm
libgcc-4.8.5-16.el7.x86_64.rpm
libgomp-4.8.5-16.el7.i686.rpm
libgomp-4.8.5-16.el7.x86_64.rpm
libstdc++-4.8.5-16.el7.i686.rpm
libstdc++-4.8.5-16.el7.x86_64.rpm
libstdc++-devel-4.8.5-16.el7.i686.rpm
libstdc++-devel-4.8.5-16.el7.x86_64.rpm
nspr-4.13.1-1.0.el7_3.x86_64.rpm
nss-softokn-freebl-3.28.3-8.el7_4.i686.rpm
nss-softokn-freebl-3.28.3-8.el7_4.x86_64.rpm
tcl-8.5.13-8.el7.i686.rpm
tcl-8.5.13-8.el7.x86_64.rpm
```

### 如果 gem 安装 redis 包时，提示 ruby 版本太低

1. 卸载 yum 过时的 ruby 环境

    ```bash
    yum remove ruby ruby-devel rubygems
    ```

2. 下载 [ruby](https://www.ruby-lang.org/en/downloads/) 源码
3. 解压，编译安装:

    ```bash
    tar -zxvf {latest_ruby.tar.gz}
    cd {latest_ruby}
    ./configure
    make
    make install
    ```

4. 下载 [gem](https://rubygems.org/pages/download) 源码
5. 解压，安装：

    ```bash
    tar -zxvf {latest_rubygem.tgz}
    cd {latest_rubygem}
    ruby setup.rb
    ```

6. 重新 `gem install redis` 或者离线下载 [gem-redis](https://rubygems.org/downloads/) 的包，本地安装
