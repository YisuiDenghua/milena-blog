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

# 使用 5.18 和 5.19 内核

可使用 Manjaro 自带的 mhwd 工具管理内核。详见 [Manjaro Kernels - Manjaro Wiki](https://wiki.manjaro.org/index.php/Manjaro_Kernels)

然而，在使用 Intel 和 NVIDIA 的机器上，5.18 及以上版本的内核无法启动。在 `/etc/default/grub` 中的 `GRUB_CMDLINE_LINUX_DEFAULT` 选项中添加一条内核命令行 `ibt=off`，然后重新生成 grub 配置文件。

```
sudo grub-mkconfig -o /boot/grub/grub.cfg 
```

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

```bash
pamac install fcitx5-im
pamac install fcitxt-rime
```

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

```bash
pamac install base-devel
```

## 安装 AUR 的软件

> 警告：AUR 不是受 Manjaro 社区支持的项目。在使用 AUR 的同时，请自担风险。

使用 `pamac` 可管理 AUR 的软件。
 
```bash
pamac search -a typora
```

上述命令可用于在 AUR 中搜索 "typora"。要安装 `typora`(AUR)，使用以下命令：

```bash
pamac build typora
```

## 指纹识别

> 提示：建议使用 GNOME + GDM, 其提供了开箱即用的指纹识别方案。

KDE Plasma 的指纹识别由 fprint 提供。

安装 `fprintd` 和 `imagemagick` 包。

```bash
pamac install fprintd imagemagick
```

在 `/etc/pam.d/system-local-login` , `/etc/pam.d/sddm` 以及 `/etc/pam.d/kde` 的开始位置添加一行：

```
auth      sufficient pam_fprintd.so
```

要想添加指纹，执行

```bash
fprintd-enroll
```


## 安装 Nix 包管理器

我们建议使用 Nix 官方的脚本来安装 Nix

```shell
sh <(curl https://mirrors.tuna.tsinghua.edu.cn/nix/latest/install) --daemon
```

同时，Manjaro 的官方软件源里提供了从 Arch Linux 移植而来的 Nix

> 提示：NixOS 社区不推荐这种安装方式

```shell
sudo pacman -S nix
sudo systemctl enable nix-daemon --now
```

## 设置软件源

```shell
nix-channel --add https://mirrors.ustc.edu.cn/nix-channels/nixpkgs-unstable
nix-channel --update
```

将以下内容写入 `/etc/nix/nix.conf`

```
substituters = https://mirror.sjtu.edu.cn/nix-channels/store https://cache.nixos.org
```


## 声明式的包管理

在非 NixOS 的发行版上，仍然可以使用 Nix 来进行声明式的包管理，但是方法和 NixOS 略有不同。同时，你无法在其他发行版上使用 NixOS 模块

## 方案一：Home Manager

### 安装 Home Manager

```shell
nix-channel --add https://github.com/rycee/home-manager/archive/master.tar.gz home-manager
nix-channel --update
nix-shell '<home-manager>' -A install
```

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

效果：

![](/images/manjaro-1.png)

如图，可以看到，我正在使用 Nix 管理软件包，以及 Manjaro 的 5.19rc7 内核。

### 实例：Git 配置

使用 Home Manager 可配置很多用户环境的软件，例如 Git，甚至 i3wm。这里以 Git 的配置为例：

```nix
  programs.git = {
    enable = true;
    userEmail = "你的电子邮件";
    userName = "你的用户名";
  };
```

> 提示：Home Manager 也有很多与 NixOS 类似的模块。你可以到 [Appendix](https://nix-community.github.io/home-manager/options.html) 查询模块。


## 方案二（不推荐，仅作参考）


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

## Nix Flake 的使用

> 警告：该部分内容不适用于 NixOS, 仅适用于非 NixOS 的发行版。
> 该部分内容默认你已经安装了 Home Manager

创建一个 `~/.config/nixpkgs/flake.nix` 文件

```nix
{
  description = "随便写，不写也行";

  inputs = {
    # Specify the source of Home Manager and Nixpkgs.
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, home-manager, ... }:
    let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      homeConfigurations.<你的用户名> = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;

        # Specify your home configuration modules here, for example,
        # the path to your home.nix.
        modules = [
          ./home.nix
        ];

        # Optionally use extraSpecialArgs
        # to pass through arguments to home.nix
      };
    };
}
```

> 提示：若使用 Home Manager 22.05, 将上述内容中 `home-manager` 的 `url` 改为 `github:nix-community/home-manager/release-22.05`

## 使用来自 NixOS CN Flake 的软件包

[NixOS CN](https://github.com/nixos-cn/flakes) 的主页已经提供了在 NixOS 上的配置方法，然而在非 NixOS 的 Linux 发行版上，配置的方式会有些不同：

在 NixOS CN 主页的示例 flake 上，有这样两个选项：

![](/images/nix2.png)

值得一提的是，在非 NixOS 的发行版上，Home Manager 无法管理系统层面的 Nix 设置。对于系统层面的 Nix 设置，需要通过编辑 `/etc/nix/nix.conf` 来实现。

在 `/etc/nix/nix.conf` 中添加以下内容：


```
substituters = https://mirror.sjtu.edu.cn/nix-channels/store https://cache.nixos.org https://nixos-cn.cachix.org
# 使用来自 SJTU 的 Nix 二进制缓存，官方二进制缓存以及 NixOS CN Flake 的二进制缓存

trusted-substituters = https://mirror.sjtu.edu.cn/nix-channels/store https://cache.nixos.org https://nixos-cn.cachix.org
# 将这些二进制缓存设置为可信的


trusted-public-keys = cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY= hydra.nixos.org-1:CNHJZBh9K4tP3EKF6FkkgeVYsS3ohTl+oS0Qa8bezVs= nixos-cn.cachix.org-1:L0jEaL6w7kwQOPlLoCR3ADx+E3Q8SEFEcB9Jaibl0Xg=
# 使用官方二进制缓存以及 NixOS CN Flake 二进制缓存的公钥

experimental-features = nix-command flakes
# 启用 Nix Flake 实验性功能

```

除此之外，不要改动 `nix.conf` 中预先生成的内容。

接下来在我们刚才创建的 `flake.nix` 基础上 加入 NixOS CN 软件：

```nix
{
  description = "Denghua";

  inputs = {
  
    # 加入 NixOS CN 仓库
    nixos-cn = {
      url = "github:nixos-cn/flakes"; # NixOS CN 的 GitHub 地址
      inputs.nixpkgs.follows = "nixpkgs"; # 强制 NixOS CN 与 nixpkgs 使用同一版本的 nixpkgs
    };
    
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, home-manager, nixos-cn, ... }: # 注意，这里将 nixos-cn 加入 outputs 集合当中
    let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      homeConfigurations.<你的用户名> = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        
        modules = [
          ./home.nix
       
          # 安装 NixOS 软件，这里以 hyfetch 为例。
          ({ ... }: {
            home.packages = 
              [
                nixos-cn.legacyPackages.${system}.hyfetch
              ];
          })

        ];

        
      };
    };
}
```

配置完成之后，在 `~/.config/nixpkgs/` 目录下使用 `home-manager switch --flake .` 命令以应用变更：

```shell
cd ~/.config/nixpkgs
home-manager switch --flake .
```

运行 `hyfetch` 以检验软件是否成功安装：

![](/images/hyfetch-1.png)
