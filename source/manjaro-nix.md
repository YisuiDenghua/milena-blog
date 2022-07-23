---
title: 在联想拯救者 Y9000P 上安装 Manjaro 并使用 Nix 管理软件包
categories: 
  - Linux
  - Nix
  - Manjaro
  - 教程
author:
  - 一穗灯花 
date: 2022-07-19 19:06:00
tags:
  - Linux
  - Nix
  - Manjaro
  - 保姆级教程
---
<h1 align="center">
  <img src="https://flatpak.org/img/distro/manjaro-0cdbc500.svg" width="200">
  <br>Manjaro<br>
</h1>

# 为什么使用 Manjaro

在这台笔记本上，我测试了很多发行版的 LiveCD，只有 Manjaro 提供了开箱即用的 Nvidia Optimus 支持，以及网卡和高刷（165fps）屏幕的支持。尽管 Manjaro 这个发行版较 Arch Linux 相比仍然存在一些问题，但不可否认的是，相比 Arch Linux 等发行版，Manjaro 提供了既易用又充足的充足的硬件支持，这也是 Manjaro 近年来热度较高的原因之一。

# 已知问题

扬声器无法使用，可使用 3.5mm 耳机或蓝牙耳机，或 HDMI 实现音频输出。

# 安装 Manjaro

参见 [Installation Guides - Manjaro Wiki](https://wiki.manjaro.org/index.php/Installation_Guides/)

# 安装后的设置

默认安装下，显卡驱动是开箱即用的。

## 包管理器

Manjaro 默认的包管理器是 `pamac` 。此外，你也可以使用 `pacman` 管理软件包，但 `pacman` 无法自动从 AUR 构建软件包。

```
可用操作:
  pamac --version     
  pamac --help, -h     [操作]
  pamac search         [选项] <软件包>
  pamac list           [选项] <软件包>
  pamac info           [选项] <软件包>
  pamac install        [选项] <软件包>
  pamac reinstall      [选项] <软件包>
  pamac remove         [选项] [软件包]
  pamac checkupdates   [选项]
  pamac update,upgrade [选项]
  pamac clone          [选项] <软件包>
  pamac build          [选项] [软件包]
  pamac clean          [选项]
```

## 安装输入法

`pamac install fcitx5-im`
`pamac install fcitxt-rime`

## 设置输入法

将以下内容添加到 `/etc/environment`，然后重新登录：

```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

## 安装 `base-devel`

要想从 AUR 构建软件，需要安装 `base-devel` 。

`pamac install base-devel`

## 安装 AUR 的软件

> 警告：AUR 不是受 Manjaro 社区支持的项目。在使用 AUR 的同时，请自担风险。

使用 `pamac` 可管理 AUR 的软件。

`pamac search -a typora` 

上述命令可用于在 AUR 中搜索 "typora"。要安装 `typora`(AUR)，使用以下命令：

`pamac build typora`

## 安装 Nix 包管理器

`pamac install nix`
`sudo systemctl enable nix-daemon --now`

## 设置软件源

`nix-channel --add https://mirrors.ustc.edu.cn/nix-channels/nixpkgs-unstable`
`nix-channel --update`

将以下内容写入 `~/.config/nix/nix.conf`

```
substituters = https://mirror.sjtu.edu.cn/nix-channels/store https://cache.nixos.org
```


## 声明式的包管理

在非 NixOS 的发行版上，仍然可以使用 Nix 来进行声明式的包管理，但是方法和 NixOS 略有不同。同时，你无法在其他发行版上使用 NixOS 模块

## 方案一：Home Manager

### 安装 Home Manager

`nix-channel --add https://github.com/rycee/home-manager/archive/master.tar.gz home-manager`
`nix-channel --update`
`nix-shell '<home-manager>' -A install`

重启

### 使用 Home Manager 管理软件包

安装  Home Manager 之后，可以使用 `~/.config/nixpkgs/home.nix` 描述用户环境：

Home Manager 会生成如下的 `home.nix`

```nix
{ config, pkgs, ... }:

{
  home.username = "你的用户名";
  home.homeDirectory = "/home/你的用户名";
  
  home.stateVersion = "22.05";

  programs.home-manager.enable = true;
}
```

使用 `home.packages` 模块来管理软件包：

```nix
  home.packages = with pkgs; [
    emacs
    git
  ];

```

完成配置后，使用 `home-manager switch` 即可使配置生效。

> 提示：Home Manager 也有很多与 NixOS 类似的模块。你可以到 [Appendix](https://nix-community.github.io/home-manager/options.html) 查询模块。


## 方案二（略不推荐）

使用 `packageOverrides` 可以进行声明式的包管理。这意味着我们可以使用一个 Nix 表达 式来描述我们想要安装的软件。例如，安装 `git` 和 `emacs`，在 `~/.config/nixpkgs/config.nix` 中加入以下内容 ：

```nix
{
  packageOverrides = pkgs: with pkgs; {
    myPackages = pkgs.buildEnv {
      name = "my-packages";
      paths = [
        git
        emacs
      ];
    };
  };
}
```

要想将它们安装到你的环境里，执行 `nix-env -iA nixpkgs.myPackages`。如果你想要从一个 `nixpkgs` 的工作副本中加载软件包，执行 `nix-env -f. -iA myPackages` 。

## NUR 与 Nix Flake 的使用

（未完待续）