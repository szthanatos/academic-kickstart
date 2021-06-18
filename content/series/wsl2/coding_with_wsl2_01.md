---
title: 开启 WSL2 安装 Ubuntu
linktitle: 配置 WSL
toc: true
type: book
date: 2020-08-22T11:37:05+08:00
lastmod: 2020-08-22T11:37:05+08:00
draft: false
weight: 10
---

## 什么是 WSL

> 适用于 Linux 的 Windows 子系统可让开发人员按原样运行 GNU/Linux 环境 - 包括大多数命令行工具、实用工具和应用程序 - 且不会产生传统虚拟机或双启动设置开销。

然而现在出的 WSL2 是基于 Hyper-V 虚拟机的...

## 什么是 WSL 2

> WSL 2 是适用于 Linux 的 Windows 子系统体系结构的一个新版本，它支持适用于 Linux 的 Windows 子系统在 Windows 上运行 ELF64 Linux 二进制文件。 它的主要目标是提高文件系统性能，以及添加完全的系统调用兼容性。

![wsl1 vs wsl2](https://i.loli.net/2021/06/17/zHdkqBjmfw6sbUr.png)

wsl1 其实感觉速度更快，和 windows 共享网络，而且没有文件系统的限制。

然而 wsl1 不是完整 Linux 内核，不支持 Docker。如果你只是要一个 Linux 环境可以考虑使用 wsl1 。

## 安装

### 启用 Windows-Subsystem-Linux

在 ` 控制面板 `-` 程序 `-  ` 启用或关闭 Windows 功能 ` 中勾选 ` 适用于 Linux 的 Windows 子系统 ` 以及 ` 虚拟机平台 `(wsl2 需要)

![启用 wsl2](https://i.loli.net/2021/06/17/NOw3m5flE8Fd62C.png)

或者通过命令行（管理员身份）执行：

```powershell
# 适用于 Linux 的 Windows 子系统
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 虚拟机平台
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

完成后需要重启电脑。

### 更新内核

目前版本的 Win10 需要手动更新 WSL2 内核，前往 [下载适用于 x64 计算机的最新 WSL2 Linux 内核](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) 下载 msi 文件手动更新。

_ARM 平台前往 [下载适用于 ARM64 计算机的最新 WSL2 Linux 内核](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_arm64.msi)_

### 设置 wsl 默认版本

通过命令行（管理员身份）执行：

```powershell
wsl --set-default-version 2
```

让未来的 Linux 都默认以 WSL2 形式安装。

现有 WSL 虚拟机也可以通过：

```powershell
# 获取现有 wsl 版本信息
wsl --list --verbose

# 将现有发行版转化为制定版本
wsl --set-version <distribution name> <versionNumber>
```

升级到 WSL2 。

## 安装 Linux 系统

当前有两种方式可选，通过微软商城一键安装或者通过wsl命令手动安装。

### a. 微软商城安装

打开 [Microsoft Store](https://aka.ms/wslstore)，搜索 wsl 即可获取可用 Linux 发行版。

现在支持的发行版有：

- Ubuntu 16.04 LTS
- Ubuntu 18.04 LTS
- Ubuntu 20.04 LTS
- openSUSE Leap 15.1
- SUSE Linux Enterprise Server 12 SP5
- SUSE Linux Enterprise Server 15 SP1
- Kali Linux
- Debian GNU/Linux
- Fedora Remix for WSL
- Pengwin
- Pengwin Enterprise
- Alpine WSL

下载完成后，点击图标进入，首次使用会需要几分钟执行安装。

完成安装后，设置 Ubuntu 用户名（非 root）及密码，正式开启 Ubuntu 系统。

![安装](https://docs.microsoft.com/zh-cn/windows/wsl/media/ubuntuinstall.png)

### b. 手动安装(推荐)

当前 Ubuntu 官方提供多个版本的镜像：

- Ubuntu Desktop——桌面版，带GUI界面和常用软件，大约3G；
- Ubuntu Server——服务器版，默认不带GUI，大约1G；
- `Ubuntu Cloud`——云版，相比服务器版更精简，大约450M；
- Ubuntu Core——为树莓派等嵌入式设备打包的特殊版本，最轻，大约300M；

这里我们选择云版镜像。

![Ubuntu Cloud](https://i.loli.net/2021/06/18/z5asZRPGOcJEFLj.jpg)

前往 [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/) 官方镜像站，
下载镜像。

以 [Ubuntu 20.10 LTS](http://cloud-images.ubuntu.com/focal/current/) 为例，
以 `wsl`为关键字，找到 AMD 或 ARM 平台对应镜像文件，下载。

![Ubuntu 20.10 LTS](https://i.loli.net/2021/06/18/ujlh7mFHDa58AUv.jpg)

之后通过命令

```powershell
wsl --import {名称} "{安装位置}" "{镜像位置}"
```

安装系统，注意路径的双引号不要省略。

## 常用操作

### wsl 命令

- `wsl --shutdown`  立即终止所有正在运行的发行版和 WSL 2 轻型工具虚拟机
- `wsl -t <发行版>` 终止指定的发行版
- `wsl -l` 列出发行版
- `wsl -l -v` 列出发行版及其版本信息
- `wsl -s <发行版>` 将发行版设为默认
- `wsl --export <发行版> <文件名>`  将发行版导出到 tar 文件
- `wsl --import <发行版> <安装位置> <文件名> [选项]`  将指定的 tar 文件作为新发行版进行导入

注意，导入导出发行版会导致无法从 Microsoft 应用管理中管理或更新。

### 网络

WSL2 相当于独立虚拟机，而 WSL1 是和 Windows 复用同一个网络的。

然后和一般宿主机虚拟机略微相反，从 Windows 可以通过 `localhost` 访问 WSL 上的服务，反之则不行。

想要访问 Windows 上的服务，可以使用命令

```bash
ip route | grep default | awk '{print $3}'
```

或者查询 `/etc/resolv.conf` 中的 `nameserver` 获取到 Windows 宿主机的 ip，

然后通过 ip 访问。

### 文件系统

WSL1 的时候，文件目录位于

- Ubuntu: `%localappdata%\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs`
- Ubuntu18.04: `%localappdata%\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs`

WSL2 可以直接通过 `\\wsl$` 在文件管理器中查看到网络位置上的 WSL 虚拟机，Ubuntu 就是 `\\wsl$\Ubuntu`。

或者你可以从 WSL2 中直接打开当前目录，直接在终端里调用 `explorer.exe .` 就行了，非常 cool~

Windows 的磁盘在 WSL 中都表示为 `/mnt/c|d|e...`，可以看，但是操作还是尽量拷贝到 WSL 目录下，因为性能损失巨大。

### 清理磁盘空间

在 Linux 下回收文件系统上所有未使用的块：

```bash
sudo fstrim /
```

对于 Win10 专业版，使用如下命令压缩 WSL 虚拟机占用空间：

```powershell
wsl --shutdown
optimize-vhd -Path "{安装位置\ext4.vhdx}" -Mode full
```

对于家庭版：

```powershell
wsl --shutdown
diskpart
# 打开新的 Diskpart 窗口
select vdisk file="{安装位置\ext4.vhdx}"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

## 移动安装位置（可选）

通过商城安装的系统无法设置安装路径，使用一段时间后体积会膨胀到 10G+ ，
可以通过开源工具 [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline)
实现将 Linux 安装到任意位置，或者将现有 Linux 子系统移动到任意位置。

如果是命令行安装，直接 `wsl --export`， `wsl --import` 就好。

### 安装 LxRunOffline

[Scoop](https://scoop.sh/) 是一个 Window 命令行包管理器，可以提供类似 apt/yum 的体验。

通过 Scoop 安装 LxRunOffline 方法如下：

```powershell
# 在 Powershell 中执行
set-executionpolicy remotesigned -scope currentuser
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
scoop bucket add extras
scoop install lxrunoffline
```

或者你可以直接下载安装二进制文件，之后运行 `regsvr32 LxRunOfflineShellExt.dll` 完成。

### 移动

```powershell
# 在 Powershell 中执行
# 查看所有已安装的发行版
lxrunoffline gd

# 移动已存在发行版， 路径格式类似于 D:\wsl\Ubuntu-18.04
LxRunOffline m -n <发行版名称> -d < 路径 >

# 等待一段时间完成移动，查看发行版当前位置
LxRunOffline di -n <发行版名称>
```

## GPU

差不多能用：

- [在 WSL 2 中启用 NVIDIA CUDA](https://docs.microsoft.com/en-us/windows/win32/direct3d12/gpu-cuda-in-wsl)
- [CUDA on Windows Subsystem for Linux (WSL) - Public Preview](https://developer.nvidia.com/cuda/wsl)

## 参考文档

- [WSL 文档](https://docs.microsoft.com/zh-cn/windows/wsl/)
- [WSL 2 常见问题解答](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-faq)
