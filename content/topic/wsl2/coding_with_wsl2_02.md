---
title: 使用 Windows-Terminal 作为终端
linktitle: 配置 Terminal
toc: true
type: book
date: 2020-08-22T16:41:01+08:00
lastmod: 2020-08-22T16:41:01+08:00
draft: false
weight: 20
---

介绍就两个字，**颜值**。

![颜值](https://raw.githubusercontent.com/szthanatos/image-host/master/windows-terminal.PNG)

安装方式同样是 [Microsoft Store](https://aka.ms/wslstore)。

默认可以使用 `win+r` 输入 `wt` 呼出。

个人使用配置如下，版本号 `1.1.2233.0`

```json
{
    // 默认打开 Ubuntu
    "defaultProfile": "{2c4de342-38b7-51cf-b940-2309a097f518}",

    "copyOnSelect": false,
    "copyFormatting": false,
    "theme": "dark",

    "profiles": {
        "defaults": {
            // 开启毛玻璃 (模糊) 效果
            "useAcrylic": true,
            // 模糊系数 0(透明)->1(不透明)
            "acrylicOpacity": 0.75,
            // zsh Powerlevel10k 所用到的字体，详见 shell 配置章节
            "fontFace": "MesloLGS NF",
            // 必须显式设置字体大小，不然初次显示会不正常
            "fontSize": 12,
            // 光标形状
            "cursorShape":"filledBox"
        },
        "list": [
            // ...
            {
                // 自己写的一个配色文件
                "colorScheme": "Sz-dark-material",
                "guid": "{2c4de342-38b7-51cf-b940-2309a097f518}",
                "hidden": false,
                "name": "Ubuntu",
                "source": "Windows.Terminal.Wsl",
                // 将起始位置改为用户根目录
                "startingDirectory": "//wsl$/Ubuntu/home/sz/"
            },
            // ...
        ]
    },
    "schemes": [
        {
            // 仿 Solarized Dark 的一个配色
            "background": "#002B36",
            "black": "#2C3E50",
            "blue": "#396FE2",
            "brightBlack": "#34495E",
            "brightBlue": "#82AAFF",
            "brightCyan": "#89DDFF",
            "brightGreen": "#C3E88D",
            "brightPurple": "#C792EA",
            "brightRed": "#FF5370",
            "brightWhite": "#FFFFFF",
            "brightYellow": "#FFCB6B",
            "cyan": "#2DDAFD",
            "foreground": "#ECEFF1",
            "green": "#9ECE58",
            "name": "Sz-material",
            "purple": "#BB80B3",
            "red": "#E54B4B",
            "white": "#D0D0D0",
            "yellow": "#FAED70"
        },
        {
            // Solarized + Campbell
            "background": "#0C0C0C",
            "black": "#2C3E50",
            "blue": "#396FE2",
            "brightBlack": "#767676",
            "brightBlue": "#82AAFF",
            "brightCyan": "#89DDFF",
            "brightGreen": "#C3E88D",
            "brightPurple": "#C792EA",
            "brightRed": "#FF5370",
            "brightWhite": "#FFFFFF",
            "brightYellow": "#FFCB6B",
            "cyan": "#2DDAFD",
            "foreground": "#ECEFF1",
            "green": "#9ECE58",
            "name": "Sz-dark-material",
            "purple": "#BB80B3",
            "red": "#E54B4B",
            "white": "#D0D0D0",
            "yellow": "#FAED70"
        }
    ],
    // ...
}
```

其他设置详见 [官方文档](https://docs.microsoft.com/zh-cn/windows/terminal/)
