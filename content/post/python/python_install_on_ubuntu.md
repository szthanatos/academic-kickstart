---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Ubuntu安装最新版本Python"
subtitle: ""
summary: ""
authors: []
tags: ["Ubuntu"]
categories: ["Python"]
date: 2018-12-01T15:59:04+08:00
lastmod: 2018-12-01T15:59:04+08:00
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

1. 添加ppa源：

    ```bash
    # 死蛇的源
    sudo add-apt-repository ppa:deadsnakes/ppa
    # 或者，jonathonf的源
    sudo add-apt-repository ppa:jonathonf/python-3.x
    ```

    如果提示没有add-apt-repository的话执行：

    ```bash
    apt install software-properties-common
    ```

2. 更新源并安装python3.x，`3.x`以你要安装的版本号为准：

    ```bash
    sudo apt update
    sudo apt install python3.x
    # 可选
    sudo apt install python3.x-dev
    sudo apt install python3.x-venv
    ```

3. 安装pip:

    ```bash
    wget https://bootstrap.pypa.io/get-pip.py
    sudo python3.x get-pip.py
    ```

4. 查看python和pip版本：

    ```bash
    # 也可以用-V
    python --version
    python3 --version
    pip --version
    pip3 --version
    ```

5. 如果关联版本不正确，备份usr/bin的软链接，重建软链接：

    ```bash
    # 设置默认python3对应python版本
    sudo ln -s /usr/bin/python3.x /usr/bin/python3
    # 设置默认pip3使用pip版本
    sudo ln -s /usr/local/bin/pip3.x /usr/bin/pip3
    ```

    注意 python默认安装位置在`/usr/bin`下，pip默认安装位置在`/usr/local/bin`下

6. pip初始化设置：

    ```bash
    mkdir ~/.pip
    touch ~/.pip/pip.conf
    # python3.6/pip18之后无需配置这个
    echo [list]>>~/.pip/pip.conf
    echo format=columns>>~/.pip/pip.conf
    # 设置默认pip源为清华大学开源镜像
    echo [global]>>~/.pip/pip.conf
    echo index-url = https://pypi.tuna.tsinghua.edu.cn/simple>>~/.pip/pip.conf
    ```

7. (可选)一键升级所有过期的包：

    ```bash
    sudo pip freeze --local | grep -v '^\-e' | cut -d = -f 1  | xargs pip install -U
    ```
