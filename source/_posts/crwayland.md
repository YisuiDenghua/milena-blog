---
title: 在 NixOS 上使用原生 Wayland 后端运行 Chromium
categories: 
  - NixOS
  - Nix
  - 教程
author:
  - 一穗灯花
  - NixOS Community
date: 2022-07-17 08:56:00
tags:
  - NixOS
  - Nix
  - 保姆级教程
---
<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>NixOS<br>
</h1>


当前版本的 Chromium 已经支持使用原生 Wayland 后端运行。在 NixOS 上，有两种实现的方式。

# 选项一：启用原生 Wayland 支持
可以通过设置特定命令行参数来启用 Chromium 的原生 Wayland 支持。将如下的代码加入 `configuration.nix` ：

```nix
  nixpkgs.config.chromium.commandLineArgs = "--enable-features=UseOzonePlatform --ozone-platform=wayland";
```

如果要让 Chromium 自动检测 Wayland 会话并以 Wayland 的方式运行，将如下的代码加入 `configuration.nix` ：

```nix
  nixpkgs.config.chromium.commandLineArgs = "--ozone-platform-hint=auto";
```

# 选项二：覆写 Chromium 包

使用 Nixpkgs 的覆写（Override）功能，可改变软件包的配置。详见 [*Nixpkgs Manual Part I Chapter 4. Overriding*](https://nixos.org/manual/nixpkgs/stable/#chap-overrides)

要启用 Chromium 的 Wayland 原生支持，将如下的代码加入 `configuration.nix` :

```nix

  environment.systemPackages = with pkgs; [
    (chromium.override {
      commandLineArgs = [
        "--enable-features=UseOzonePlatform"
        "--ozone-platform=wayland"
      ];
    })
  ];
   
```

如果要让 Chromium 自动检测 Wayland 会话并以 Wayland 的方式运行，将如下的代码加入 `configuration.nix` ：

```nix

  environment.systemPackages = with pkgs; [
    (chromium.override {
      commandLineArgs = [
        "--ozone-platform-hint=auto"
      ];
    })
  ];
   
```

# 对于 Google Chrome 以及基于 Electron 的软件

> 提示：本文中提到的方式亦可用于 Chrome 和其他基于 Electron 的软件。Google Chrome 在 Nixpkgs 中的包名是 `google-chrome` 。可以到 [search.nixos.org](https://search.nixos.org) 查询软件包的包名。