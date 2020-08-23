---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: 使用Zsh作为默认shell
linktitle: Shell配置
toc: true
type: docs
date: 2020-08-22T16:41:05+08:00
lastmod: 2020-08-22T16:41:05+08:00
draft: false
menu:
  wsl2:
    weight: 5

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 5
---

// TODO 吹一下zsh

## 安装zsh

```bash
# 安装zsh
sudo apt install zsh -y

# 修改默认shell为zsh
sudo chsh -s /bin/zsh
```

重启wsl，或者干脆重启电脑生效。

## 安装oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

可能需要代理。

## 安装zsh插件

```bash
# 安装powerlevel10k主题
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k

# 安装语法高亮zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 安装自动建议zsh-autosuggestions
git clone git://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

从win10下载并安装以下字体，以获得powerlevel10k主题更好的显示效果：

![powerlevel10k](https://raw.githubusercontent.com/romkatv/powerlevel10k-media/master/prompt-styles-high-contrast.png)

- [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
- [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
- [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
- [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

## 配置zsh

在`.zshrc`中编辑以下部分：

```properties
# （可选）允许其他用户共用你的配置不弹出警告
ZSH_DISABLE_COMPFIX="true"

# 默认情况下，zsh会试图在低优先级运行后台任务，但是Windows不允许，所以将以下内容添加到.zshrc文件开头可以改变zsh的行为
case $(uname -a) in
  *Microsoft*) unsetopt BG_NICE ;;
esac

# 启用powerlevel10k主题
ZSH_THEME="powerlevel10k/powerlevel10k"

# 启用插件
plugins=(
  # 按z+{模糊文件夹名称}快速跳转
  z
  # git补全
  git
  # 忘了加sudo的时候按两下esc
  sudo
  # docker补全
  docker
  docker-compose
  # 按x解压任意类型压缩包
  extract
  # 彩色man手册
  colored-man-pages
  zsh-autosuggestions
  zsh-syntax-highlighting
)

# 配置终端颜色
export TERM="xterm-256color"
```

输入`source .zshrc`让配置生效，

输入`p10k configure`配置Powerlevel10k主题设置。

![p10k configure](https://raw.githubusercontent.com/romkatv/powerlevel10k-media/master/configuration-wizard.gif)

完成后，拷贝`.zshrc`和`.p10k.zsh`文件到root用户根目录下，让root也使用相同配置，当然也可以分别设置。

## 一键启用/停用代理

Linux下配置代理本来就略麻烦，Wsl2又是完整的虚拟机，每次重启内部网络都会重新生成...换句话说每次Windows 和Linux 的内网IP都在变化...

不过还是能配置好的。

将以下内容加入到你的`.zshrc`文件中，将`20809`改为你的代理的端口：

```shell
proxy() {
  local host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
  export ALL_PROXY="http://${host_ip}:20809"
  export all_proxy="http://${host_ip}:20809"
  echo -e "Acquire::http::Proxy \"http://${host_ip}:20809\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
  echo -e "Acquire::https::Proxy \"http://${host_ip}:20809\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
  curl ip.sb
}

unproxy() {
  unset ALL_PROXY
  unset all_proxy
  sudo sed -i -e '/Acquire::http::Proxy/d' /etc/apt/apt.conf
  sudo sed -i -e '/Acquire::https::Proxy/d' /etc/apt/apt.conf
  curl ip.sb
}
```

执行`proxy`即可开启代理，`wget`，`curl`，`git`，`apt` 都会走代理。

执行`unproxy`关闭。

## 参考

- [Oh My ZSH官网](https://ohmyz.sh/)
- [powerlevel10k 项目主页](https://github.com/romkatv/powerlevel10k)
