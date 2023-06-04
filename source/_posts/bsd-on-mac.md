---
title: 笔记：在旧款 MacBook 上安装 OpenBSD 及配置图形界面，输入法
categories:
  - OpenBSD
  - 教程
author:
  - 一穗灯花
date: 2023-06-04 10:54:00
tags:
  - OpenBSD
---

<h1 align="center">
  <img src="https://static.wikia.nocookie.net/logopedia/images/3/34/800px-OpenBSD_Logo_-_Cartoon_Puffy_with_textual_logo_below.svg.png/revision/latest?cb=20230127155203" width="200">
  <br>OpenBSD<br>
</h1>

> 结论：除摄像头之外的所有硬件均可正常工作。目前正在寻找摄像头的解决方案。

我手里有一台 2015 年的 MacBookPro12,1，曾经的顶配，到了如今已经显得有些年迈。在它停止接收 macOS 更新之后，我决定为它安装 OpenBSD 这一轻量极系统。

## 获取 OpenBSD

可以从清华镜像站获取 OpenBSD 安装镜像。由于我使用的是 OpenBSD 7.3,下载的是 install73.img

https://mirrors.tuna.tsinghua.edu.cn/OpenBSD/7.3/amd64/

## 添加无线固件

在 OpenBSD 安装后，大多数驱动固件都可以通过 `fw_update`(8) 自动安装。但网卡是一种较为特殊的硬件。如果网卡不能工作，我们将不能连接到互联网，也无法通过 `fw_update`(8) 安装固件。在安装之前，我们需要找一个已经安装了 OpenBSD 的电脑（或者虚拟机），将 install.img 拷入其中。

```
# vnconfig install73.img
vnd0
# mount /dev/vnd0a /mnt
# fw_update -Fv -p /mnt bwfm #我的笔记本搭载的是博通网卡，需要 bwfm。
# umount /mnt
# vnconfig -u vnd0
```

如果你不确定你的网卡是什么，可以先启动一个 OpenBSD USB,然后使用 `ifconfig`(8) 来查看你的网卡型号。

![](/images/bwfm.png)

可在 https://www.openbsdhandbook.com/networking/wireless/ 查看 OpenBSD 所支持的所有网卡。

## 安装 OpenBSD

按照向导指示安装既可，此过程不赘述。如果要启用图形界面，安装时将 X 相关的选项选为 "yes" 。

## 连接 WiFi

使用 `ifconfig`(8) 连接到网络，此过程需要 root 权限。

```
$ doas ifconfig bwfm0 nwid 网络名称 wpakey 密码
```

## 图形界面

我使用的是 CWM(1) 窗口管理器，它是 OpenBSD 的一部分，不需要单独安装。

CWM(1) 可使用 `.cwmrc` 和 `.xsession` 或 `.xinitrc` 来配置。如果你使用 `xenodm`(1) 等显示管理器进入图形界面，则使用 `.xsession`；如果使用 `startx`(1) ，则要配置 `.xinitrc`。我这里以前者情况为例，贴上一个我自己使用的 cwm 配置。

我在 GitHub 上放了一个我目前使用的 cwm 配置。https://github.com/YisuiDenghua/cwm-configs-openbsd/tree/main

目前我没有找到使用 MacBook 的功能键调整屏幕亮度与音量的方式。目前只能使用 `xbacklight`(1) 和 `pavucontrol` 调节。

## 输入法

安装 `fcitx` `fcitx-configtool-qt` `fcitx-gtk` `fcitx-qt`

安装中文字体 `noto-cjk`

安装中文方案 `fcitx-chinese-addons` `fcitx-table-extra`，此方案包含了拼音，五笔等。

# 科学上网

编译安装 v2ray 与 v2raya.
