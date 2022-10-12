---
title: 杂谈：让 NixOS 的 GNOME X11 会话实现 150% 缩放
categories: 
  - NixOS
  - 教程
#pic: https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/NixOS_logo.svg/1280px-NixOS_logo.svg.png
author: 
 - 一穂灯花
date: 2022-10-12 22:53:00
tags: 
 - NixOS
 - Linux
---

<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>NixOS<br>
</h1>

<div class="info">

# 前言

这学期灯花购入了一台联想拯救者 Y9000P 并装了个 NixOS。除了声卡和睡眠休眠有问题之外，使用起来还是不错的，最新的内核也支持 12 代 i7 CPU 的大小核调度。但……还有一点小问题：就是这台笔记本的屏幕默认是 150% 缩放的，然而 GNOME 只支持诸如 100%，200% 的整数倍缩放，并不支持 150% 的分数倍缩放。导致咱被 KDE 所「绑架」了长达半年多（bushi

![](/images/gnome-prepatch.jpg)

如图，只有 100, 200, 300。在这屏幕上，100% 的缩放看起来太小，200% 又太大，GNOME 的 Wayland 虽支持非整数倍缩放，但会导致字体发虚。然而 KDE 却能很好地支持 150% 非整数倍缩放，导致咱只能忍痛告别了陪伴咱多年的 GNOME ，投奔了 KDE（QAQ

半年后，偶然在 [Arch Wiki](https://wiki.archlinux.org/title/HiDPI#Xorg) 上看到了如下内容：

> 「Ubuntu 提供了一个补丁以在 GNOME 设置选项中使用 Randr。这个补丁也为 mutter-x11-scaling （AUR） 所提供。在安装之后，使用
> 
> ```
> # 注意，该命令在 NixOS 上不可用
> gsettings set org.gnome.mutter experimental-features "['x11-randr-fractional-scaling']"
> ```
> 以启用非整数倍缩放的功能。」

于是我打开了它的 PKGBUILD 文件并一探究竟：


```
_scaling_commit=c3ca443d49da639e73f27ab92a758a41259ba732 # Commit c3ca443d
_commit=4b35c269c7ad2515f91d2d3ccaea7526e0cb5f97  # tags/42.5^0
source=("git+https://gitlab.gnome.org/GNOME/mutter.git#commit=$_commit"
	"x11-Add-support-for-fractional-scaling-using-Randr.patch::https://salsa.debian.org/gnome-team/mutter/-/raw/$_scaling_commit/debian/patches/ubuntu/x11-Add-support-for-fractional-scaling-using-Randr.patch"
	"Support-Dynamic-triple-double-buffering.patch::https://raw.githubusercontent.com/puxplaying/mutter-x11-scaling/11525ce32c13b542bbfcb7f43a76d51111077cea/Support-Dynamic-triple-double-buffering.patch")
```

上段内容是这个 PKGBUILD 脚本异于原版的特点，相比于原版，它在源代码中增加了两个补丁（patch）文件，并重新打包了 mutter。

然而 Nix 提供了一个叠层（Overlay）功能，可以用于修改 Nixpkgs 中软件包的构建。得益于叠层（Overlay），可以在不重新打包 mutter 的情况下修改这个包的构建。

# 正文

正片开始！

首先在 /etc/nixos/ 下新建一个叠层（Overlay）文件，例如 `fr.nix`，然后加入如下内容：

```
final: prev: {
  gnome = prev.gnome.overrideScope'
    (final': prev': {
      mutter = prev'.mutter.overrideAttrs (old: {
        patches = (old.patches or []) ++ [
          (final.fetchpatch {
	    url = "https://raw.githubusercontent.com/puxplaying/mutter-x11-scaling/11525ce32c13b542bbfcb7f43a76d51111077cea/Support-Dynamic-triple-double-buffering.patch";
	    sha256 = "sha256-dTv7tNyTcOj1nJLDLfccWuZuMs62VtowZ2pz8QaRW9A=";
	  })
	  (final.fetchpatch {
	    url = "https://salsa.debian.org/gnome-team/mutter/-/raw/c3ca443d49da639e73f27ab92a758a41259ba732/debian/patches/ubuntu/x11-Add-support-for-fractional-scaling-using-Randr.patch";
	    sha256 = "sha256-ut78BQemEF8T3189c9KWGrJ2mh8mFkgUxpad0ogNkFI=";
	  })
        ];
      });
    });
}
```

然后在 `configuration.nix` 中启用该叠层（Overlay）：

```
  nixpkgs.overlays = [
    (import ./fr.nix)
  ];

```

以及启用 GNOME X11 的非整数倍缩放功能：

```
  services.xserver.desktopManager.gnome = {
    enable = true;
    extraGSettingsOverridePackages = [ pkgs.gnome.mutter ];
    extraGSettingsOverrides = ''
      [org.gnome.mutter]
      experimental-features=['x11-randr-fractional-scaling']
    '';
  };
```

然后，使用 `nixos-rebuild switch` 重构 NixOS，并重新登录 GNOME X11。

# 结语

大功告成啦！

重新打开 GNOME 的显示设置，可以看到，出现了很多非整数倍的缩放选项：

![](/images/gnome-postpatch.png)

