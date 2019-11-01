---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "基于Rancher HA搭建容器云平台"
subtitle: ""
summary: ""
authors: []
tags: ["rancher"]
categories: []
date: 2019-09-26T22:58:39+08:00
lastmod: 2019-09-26T22:58:39+08:00
featured: false
draft: true

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

## 硬件需求

rancher ha推荐的架构是单独搭建一个K8s集群部署Rancher，然后再用这个Rancher去管理其他的K8s集群。

![推荐架构](/img/rancher2ha.png)

rancher集群的配置和能管理的集群规模之间的关系如下：

| 规模         | 集群      | 节点       | 虚拟CPU核数 | 内存        |
| ------------ | --------- | ---------- | ----------- | ----------- |
| **S**mall    | 最多5个   | 最多50个   | 2           | 8 GB        |
| **M**iddle   | 最多15个  | 最多200个  | 4           | 16 GB       |
| **L**arge    | 最多50个  | 最多500个  | 8           | 32 GB       |
| **X-L**arge  | 最多100个 | 最多1000个 | 32          | 128 GB      |
| **XX-L**arge | 100+      | 1000+      | 联系Rancher | 联系Rancher |

单节点部署的Rancher只支持中小规模(**S**,**M**)的集群。

我这次部署的机器配置如下：

| 编号        | 地址          | 虚拟CPU核数 | 内存  |
| ----------- | ------------- | ----------- | ----- |
| loadbalance | 192.168.1.100 | 2           | 8G    |
| server-01   | 192.168.1.101 | 4           | 16 GB |
| server-02   | 192.168.1.102 | 4           | 16 GB |
| server-03   | 192.168.1.103 | 4           | 16 GB |

操作系统均为`RancherOS 1.54`(`Console`是`Ubuntu`，因为`Longhorn`只支持这个)，Docker版本`18.09`。

## 工具

在任意一台server上准备以下工具：

| 工具    | 说明                                            | 版本                                                                                               |
| ------- | ----------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| kubectl | Kubernetes命令行工具                            | [kubernetes-v1.16.0](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.16.md#v1160) |
| rke     | Rancher出品的用于构建Kubernetes集群的命令行工具 | [Release v0.2.8](https://github.com/rancher/rke/releases/tag/v0.2.8)                               |
| helm    | k8s包管理工具                                   | [Helm v2.14.3](https://github.com/helm/helm/releases/tag/v2.14.3)                                  |

- `kubectl`安装：

  ```bash
  wget https://dl.k8s.io/v1.16.0/kubernetes-client-linux-amd64.tar.gz
  tar -zxvf kubernetes-client-linux-amd64.tar.gz
  sudo ln -s $(pwd)/kubernetes/client/bin/kubectl /usr/local/bin/kubectl
  ```

- `rke`安装：

  ```bash
  wget https://github.com/rancher/rke/releases/download/v0.2.8/rke_linux-arm64
  sudo ln -s $(pwd)/rke_linux-amd64 /usr/local/bin/rke
  ```

- `helm`安装：

  ```bash
  wget https://get.helm.sh/helm-v3.0.0-beta.3-linux-amd64.tar.gz
  tar -zxvf helm-v2.14.3-linux-amd64.tar.gz
  sudo ln -s $(pwd)/linux-amd64/helm /usr/local/bin/helm
  ```

## 过程

## 
