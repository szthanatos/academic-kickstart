---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: 使用Docker作为开发环境
linktitle: Docker环境配置
toc: true
type: docs
date: 2020-08-22T18:07:41+08:00
lastmod: 2020-08-22T18:07:41+08:00
draft: false
menu:
  wsl2:
    weight: 8

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 8
---

嗯，是的，其实可以直接在Windows下运行Docker而无需Wsl。但是Wsl2会提供更好的性能——

如果开启Wsl2，Docker会默认把自己的文件系统迁移到wsl上（因为在Win下的本质也是开了个Hyper-V的虚拟机在运行）

——另一方面，咱们前面配好的终端环境也比Windows的终端好使...

看到说Powershell好的人有，但是真拿它写东西的人好像没见到过...

再一个是Docker Desktop可以访问到Windows下的文件，而如果直接在wsl2中安装Docker就没有这份福利了...

所以Docker Desktop + Wsl2是最合适的一个组合。

## 安装

1. 下载安装[Docker Desktop](https://download.docker.com/win/edge/Docker%20Desktop%20Installer.exe) edge版本
2. 打开设置，检查`General`中`use the WSL2 base egine`应该是默认勾选的状态；
3. 在`Resources`-`WSL INTEGRATION`中检查`Enable integration with my default WSL Distro`应该是勾选状态，并将Linux发行版置为`Enable`状态;
4. 点击`Apply & Restart`，完成~

Docker Desktop重启之后，在任意终端(Windows & Linux)都可以执行Docker命令啦~

## 配置开发环境

在`设置`-`构建、执行、部署`下找到`Docker`配置，可以直接连接Docker for window：

![连接docker](https://raw.githubusercontent.com/szthanatos/image-host/master/pycharm-docker.png)

我惯用的开发环境Dockerfile如下：

```dockerfile
FROM python:3.8-buster
LABEL author="Lex Wayne"

WORKDIR /app
COPY . /app
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple &&\
    pip install -U pip &&\
    pip install --no-warn-script-location -r /requirements.txt
```

完成Docker配置后会在左下方底栏新增一个`Services`，没有的话从顶部`视图`-`工具窗口`中也能找到，

双击连接Docker，点击侧边绿色三个箭头的图表即可使用Dockerfile部署。

![dockerfile部署](https://raw.githubusercontent.com/szthanatos/image-host/master/pycharm-dockerfile.png)

## 参考

- Docker官方文档: [Docker Desktop WSL 2 backend](https://docs.docker.com/docker-for-windows/wsl/)
- Vs Code团队博客: [Using Docker in WSL 2](https://code.visualstudio.com/blogs/2020/03/02/docker-in-wsl2)
