---
title: 使用 Flakes 配置 NixOS
categories: 
  - NixOS
  - Flakes
  - 教程
#pic: https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/NixOS_logo.svg/1280px-NixOS_logo.svg.png
author: 
 - 一穂灯花
 - NixOS Chinese Community
date: 2022-08-08 02:42:54
tags: 
 - NixOS
 - Linux
---

<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>NixOS<br>
</h1>

<div class="info">

> 不能再这样了... 我知道，我是个爱哭的人。但以后可不能像今天这样哭下去了。毕竟，我一穗灯花，可是号称文韬武略，天下第一的人，至少要做出一份坚强的样子啊。如果像这样软弱的话，怎么去保护大家……
> 
> —— 一穗灯花，2022 年 8 月 8 日

# Flake 简介

Flake 是 Nix 的一个实验性功能，它支持一系列 Nix 命令，以及在配置文件中定义软件源 URL 版本，使其实现 Nix 的可复现性，便于批量部署。很多第三方的软件仓库和模块（如 NixOS-CN）也是利用 Flake 实现的。

# 启用 Nix Flake

在 `configuration.nix` 中添加如下的内容：

```nix
  nix = {
    package = pkgs.nixUnstable;
    extraOptions = ''
      experimental-features = nix-command flakes
    '';
  };
```

# 配置 Nix Flake

在 `/etc/nixos/` 下新建一个名为 `flake.nix` 的文件。并向编辑文件，写入如下内容：

```nix
{
  description = "随便写，不写也行";

  # 输入配置，即软件源
  inputs = {
    # 使用 Nixpkgs，即 NixOS 官方软件源
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };

  # 输出配置，即 NixOS 系统配置
  outputs = { self, nixpkgs }:
    let system = "x86_64-linux"; # 可替换为你的系统平台，如 arm64-linux
    in {
      nixosConfigurations."你的主机名" = nixpkgs.lib.nixosSystem {
        inherit system;
        modules = [
          ./configuration.nix
        ];
      };
    };

    # 你也可以在同一份 Flake 中定义好几个系统，NixOS 会根据主机名 Hostname 决定用哪个
    # nixosConfigurations."nixos2" = nixpkgs.lib.nixosSystem {
    #   system = "x86_64-linux";
    #   modules = [
    #     ./configuration2.nix
    #   ];
    # };
    #
    # 参考 [Lantian 的博客](https://lantian.pub/article/modify-website/nixos-initial-config-flake-deploy.lantian/)

}
```

配置完成后，使用 `nixos-rebuild` 更新 NixOS 以及使用 `nix flake` 命令更新 flake：

```bash
sudo nixos-rebuild switch
sudo nix flake update
```
# 使用 NixOS-CN 及 NUR 第三方仓库

在启用 Flake 功能之后，在 `flake.nix` 中添加如下内容：

```

{
  description = "随便写，不写也行";


  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

    # 添加 NixOS CN 仓库
    nixos-cn.url = "github:nixos-cn/flakes";
    # 强制 nixos-cn 和该 flake 使用相同版本的 nixpkgs
    nixos-cn.inputs.nixpkgs.follows = "nixpkgs";
    # 添加 NUR 仓库
    nur.url = "github:nix-community/NUR";
    # 强制 NUR 和该 flake 使用相同版本的 nixpkgs
    nur.inputs.nixpkgs.follows = "nixpkgs";


  };

  # 在 outputs 中添加 nixos-cn 以及 nur

  outputs = { self, nixpkgs, nixos-cn, nur }:
    let system = "x86_64-linux";
    in {
      nixosConfigurations."你的主机名" = nixpkgs.lib.nixosSystem {
        inherit system;
        modules = [
          ./configuration.nix
          
          # NixOS CN
          ({ ... }: {
            # 使用 nixos-cn flake 提供的包
            environment.systemPackages =
              [ nixos-cn.legacyPackages.${system}.wechat-uos];
            # 使用 nixos-cn 的 binary cache
            nix.binaryCaches = [
              "https://nixos-cn.cachix.org"
            ];
            nix.binaryCachePublicKeys = [ "nixos-cn.cachix.org-1:L0jEaL6w7kwQOPlLoCR3ADx+E3Q8SEFEcB9Jaibl0Xg=" ];

            imports = [
              # 将nixos-cn flake提供的registry添加到全局registry列表中
              # 可在`nixos-rebuild switch`之后通过`nix registry list`查看
              nixos-cn.nixosModules.nixos-cn-registries

              # 引入nixos-cn flake提供的NixOS模块
              nixos-cn.nixosModules.nixos-cn
            ];
          })

          # 启用 NUR
          nur.nixosModules.nur
          
          ({ config, ... }: {
            # 使用 NUR 提供的包，这里以 hyfetch 为例
            environment.systemPackages = [ 
              config.nur.repos.YisuiMilena.hyfetch
            ];
          })

        ];
      };
    };
}
```
运行 `hyfetch` 以检验软件是否正确安装：

![](/images/hyfetch.png)
