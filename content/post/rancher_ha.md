---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "基于 Rancher HA 搭建容器云平台"
subtitle: ""
summary: ""
authors: []
tags: ["rancher"]
categories: []
date: 2019-09-26T22:58:39+08:00
lastmod: 2019-09-26T22:58:39+08:00
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

## 硬件需求

rancher ha 推荐的架构是单独搭建一个 K8s 集群部署 Rancher，然后再用这个 Rancher 去管理其他的 K8s 集群。

![推荐架构](https://i.loli.net/2021/06/17/lD9wLiNnOgrVhIs.png)

rancher 集群的配置和能管理的集群规模之间的关系如下：

| 规模         | 集群        | 节点         | 虚拟 CPU 核数 | 内存         |
| ------------ | ----------- | ------------ | ------------- | ------------ |
| **S**mall    | 最多 5 个   | 最多 50 个   | 2             | 8 GB         |
| **M**iddle   | 最多 15 个  | 最多 200 个  | 4             | 16 GB        |
| **L**arge    | 最多 50 个  | 最多 500 个  | 8             | 32 GB        |
| **X-L**arge  | 最多 100 个 | 最多 1000 个 | 32            | 128 GB       |
| **XX-L**arge | 100+        | 1000+        | 联系 Rancher  | 联系 Rancher |

单节点部署的 Rancher 只支持中小规模 (**S**,**M**) 的集群。

我这次部署的机器配置如下：

| 主机名      | 虚拟 CPU 核数 | 内存  |
| ----------- | ------------- | ----- |
| loadbalance | 2             | 8G    |
| server-01   | 4             | 16 GB |
| server-02   | 4             | 16 GB |
| server-03   | 4             | 16 GB |

一个负载均衡服务器，3 个服务器组成 K8S 集群。

操作系统均为 `RancherOS 1.54`(`Console` 是 `Ubuntu`，因为 `Longhorn` 只支持这个)，Docker 版本 `18.09`。

## 0. 工具准备

在任意一节点 (我这里都是在 `server-01` 上) 上准备以下工具：

| 工具    | 说明                                               | 版本                                                                                               |
| ------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| kubectl | Kubernetes 命令行工具                              | [kubernetes-v1.16.0](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.16.md#v1160) |
| rke     | Rancher 出品的用于构建 Kubernetes 集群的命令行工具 | [Release v0.2.8](https://github.com/rancher/rke/releases/tag/v0.2.8)                               |
| helm    | Kubernetes 包管理工具                              | [Helm v2.14.3](https://github.com/helm/helm/releases/tag/v2.14.3)                                  |

- `kubectl` 安装：

  ```bash
  wget https://dl.k8s.io/v1.16.0/kubernetes-client-linux-amd64.tar.gz
  tar -zxvf kubernetes-client-linux-amd64.tar.gz
  sudo ln -s $(pwd)/kubernetes/client/bin/kubectl /usr/local/bin/kubectl
  ```

- `rke` 安装：

  ```bash
  wget https://github.com/rancher/rke/releases/download/v0.2.8/rke_linux-arm64
  sudo ln -s $(pwd)/rke_linux-amd64 /usr/local/bin/rke
  ```

- `helm` 安装：

  ```bash
  wget https://get.helm.sh/helm-v3.0.0-beta.3-linux-amd64.tar.gz
  tar -zxvf helm-v2.14.3-linux-amd64.tar.gz
  sudo ln -s $(pwd)/linux-amd64/helm /usr/local/bin/helm
  ```

## 1. 配置 Nginx 负载均衡

在 `loadbalance` 主机上编写 nginx.conf 配置文件：

```properties
worker_processes 4;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

stream {
    upstream rancher_servers_http {
        least_conn;
        server <IP_NODE_1>:80 max_fails=3 fail_timeout=5s;
        server <IP_NODE_2>:80 max_fails=3 fail_timeout=5s;
        server <IP_NODE_3>:80 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     80;
        proxy_pass rancher_servers_http;
    }

    upstream rancher_servers_https {
        least_conn;
        server <IP_NODE_1>:443 max_fails=3 fail_timeout=5s;
        server <IP_NODE_2>:443 max_fails=3 fail_timeout=5s;
        server <IP_NODE_3>:443 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     443;
        proxy_pass rancher_servers_https;
    }
}
```

即使用 stream 方式让 nginx 转发 80/443 端口的 http/https 流量。

启动 nginx：

```bash
docker run -d \
       --name lb-nginx \
       --restart =unless-stopped \
       -p 80:80 \
       -p 443:443 \
       -v /nginx.conf:/etc/nginx/nginx.conf \
       nginx:1.14
```

## 2. 使用 RKE 安装 K8S

三个作为 rancher-server 的服务器需要配置相互免密。`ssh-keygen` 生成密钥附加到 `authorized_keys` 上，不赘述。

在 `server-01` 节点，编写 `rancher-cluster.yml`，告诉 rke 要如何创建集群：

```yaml
nodes:
  - address: <IP_NODE_1>
    internal_address: <IP_NODE_1>
    user: rancher
    role: [controlplane,worker,etcd]
  - address: <IP_NODE_2>
    internal_address: <IP_NODE_2>
    user: rancher
    role: [controlplane,worker,etcd]
  - address: <IP_NODE_3>
    internal_address: <IP_NODE_3>
    user: rancher
    role: [controlplane,worker,etcd]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h
```

节点的配置项中，

`internal_address` 非必填，如果没有内网 IP 的话可以删去。

如果没有配置 `ssh_key_path`，则会默认使用 `$HOME/.ssh/id_rsa` 建立连接。

执行

```bash
rke up --config ./rancher-cluster.yml
```

完成创建，中间如果失败了可以多执行几次，直到最后看到消息

`Finished building Kubernetes cluster successfully.`

说明安装完毕。

K8S 创建成功后会在根目录生成集群信息 `rancher-cluster.rkestate` 和配置文件 `kube_config_rancher-cluster.yml`，

这两个文件包含访问 K8S 的凭据。

## 3. 初始化 helm

helm 是由客户端 helm 和服务端 tiller 组成，我们之前安装了 helm 可以调用 helm 命令了。

为了保存和管理 helm 软件包 (helm charts)，我们还需要在本地启动一个服务端。

```bash
# 创建名为 tiller 的 serviceaccount
kubectl -n kube-system create serviceaccount tiller

# 授予 tiller 帐户对集群的访问权限
kubectl create clusterrolebinding tiller \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:tiller

# 安装 tiller，官方国内用的是阿里的源
# helm init --service-account tiller --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:<tag>
helm init \
     --service-account tiller \
     --upgrade \
     --tiller-image gcr.azk8s.cn/kubernetes-helm/tiller:v2.14.3 \
     --stable-repo-url https://mirror.azure.cn/kubernetes/charts/

# 测试是否安装成功
kubectl -n kube-system rollout status deploy/tiller-deploy
helm version
```

## 4. 配置 ca 证书

rancher 支持三种来源的证书，rancher 自生成 / 来自 Let’s Encrypt 的 / 来自文件的。

前两种都需要额外安装 CERT-MANAGER。

这里我们采用第一种方式，依据 [cert-manager 官方文档](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#installing-with-helm)：

```bash
# Install the CustomResourceDefinition resources separately
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.11.0 \
  jetstack/cert-manager
```

使用 `kubectl get pods --namespace cert-manager` 测试是否安装成功，

应该能看到 `cert-manager`，`cert-manager-webhook`，`cert-manager-cainjector` 运行中。

注意 cert-manager 使用的镜像来自 `quay.io`，可以编辑 charts 中的 `values.yaml` 来修改镜像源，或者提前从国内源下载好镜像放到服务器上。

## 5. 安装 rancher

```bash
# 在 helm 中添加 rancher 源，建议使用 stable
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update

# 安装 rancher
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=<你的域名>
```

等待一段时间，可以运行 `kubectl -n cattle-system rollout status deploy/rancher` 查看安装进度。

## 6. 添加主机别名

由于没有内部 DNS 服务器，我们还需要为 Agent Pod 添加主机别名 (/etc/hosts)。

不然

> K8S 集群运行起来之后，因为 `cattle-cluster-agent Pod` 和 `cattle-node-agent` 无法通过 DNS 记录找到 `Rancher Server URL`, 最终导致无法通信。

所以我们需要 (以下步骤直接复制于 [Rancher2.0-CN 文档](https://www.rancher.cn/docs/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/rancher-install/#6-%E5%8F%AF%E9%80%89-%E4%B8%BAagent-pod%E6%B7%BB%E5%8A%A0%E4%B8%BB%E6%9C%BA%E5%88%AB%E5%90%8D-etc-hosts))：

> 1. `cattle-cluster-agent Pod` 和 `cattle-node-agent` 需要在 `LOCAL` 集群初始化之后才会部署，所以先通过 `Rancher Server URL` 访问 Rancher Web UI 进行初始化。
>
> 2. 执行以下命令为 Rancher Server 容器配置 hosts:
>
>    ```bash
>    #指定 kubectl 配置文件
>    export kubeconfig=xxx/xxx/xx.kubeconfig.yml
>
>    kubectl --kubeconfig=$kubeconfig -n cattle-system \
>        patch deployments rancher --patch '{
>            "spec": {
>                "template": {
>                    "spec": {
>                        "hostAliases": [
>                            {
>                                "hostnames":
>                                [
>                                    "xxx.cnrancher.com"
>                                ],
>                                    "ip": "192.168.1.100"
>                            }
>                        ]
>                    }
>                }
>            }
>        }'
>    ```
>
> 3. 通过 `Rancher Server URL` 访问 Rancher Web UI，设置用户名密码和 `Rancher Server URL` 地址，然后会自动登录 Rancher Web UI；
>
> 4. 在 Rancher Web UI 中依次进入 `local 集群 / system 项目`，在 `cattle-system` 命名空间中查看是否有 `cattle-cluster-agent Pod` 和 `cattle-node-agent` 被创建。如果有创建则进行下面的步骤，没有创建则等待；
>
> 5. cattle-cluster-agent pod
>
>    ```bash
>    export kubeconfig=xxx/xxx/xx.kubeconfig.yml
>
>    kubectl --kubeconfig=$kubeconfig -n cattle-system \
>    patch deployments cattle-cluster-agent --patch '{
>        "spec": {
>            "template": {
>                "spec": {
>                    "hostAliases": [
>                        {
>                            "hostnames":
>                            [
>                                "demo.cnrancher.com"
>                            ],
>                                "ip": "192.168.1.100"
>                        }
>                    ]
>                }
>            }
>        }
>    }'
>    ```
>
> 6. cattle-node-agent pod
>
>    ```bash
>    export kubeconfig=xxx/xxx/xx.kubeconfig.yml
>
>    kubectl --kubeconfig=$kubeconfig -n cattle-system \
>    patch  daemonsets cattle-node-agent --patch '{
>        "spec": {
>            "template": {
>                "spec": {
>                    "hostAliases": [
>                        {
>                            "hostnames":
>                            [
>                                "xxx.rancher.com"
>                            ],
>                                "ip": "192.168.1.100"
>                        }
>                    ]
>                }
>            }
>        }
>    }'
>    ```

这几步花的时间比较长，需要耐心等待。安装过程到此结束。
