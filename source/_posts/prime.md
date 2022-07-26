---
title: NixOS NVIDIA PRIME 配置笔记
categories: 
  - Linux
  - Nix
  - NixOS
  - 教程
author:
  - 一穗灯花 
date: 2022-07-19 19:06:00
tags:
  - Linux
  - Nix
  - NixOS
  - 保姆级教程
---

<p align="center">
  <a href="https://nixos.org#gh-light-mode-only">
    <img src="https://raw.githubusercontent.com/NixOS/nixos-homepage/master/logo/nixos-hires.png" width="500px" alt="NixOS logo"/>
  </a>
  <a href="https://nixos.org#gh-dark-mode-only">
    <img src="https://raw.githubusercontent.com/NixOS/nixos-artwork/master/logo/nixos-white.png" width="500px" alt="NixOS logo"/>
  </a>
</p>

我的硬件是 Lenovo Legion Y9000P，Intel Iris Xe 核显和 NVIDIA RTX 3060 独显。
# Offload 模式

在 `configuration.nix` 中添加：

```nix
{ config, pkgs, ... }:
let
  nvidia-offload = pkgs.writeShellScriptBin "nvidia-offload" ''
    export __NV_PRIME_RENDER_OFFLOAD=1
    export __NV_PRIME_RENDER_OFFLOAD_PROVIDER=NVIDIA-G0
    export __GLX_VENDOR_LIBRARY_NAME=nvidia
    export __VK_LAYER_NV_optimus=NVIDIA_only
    exec "$@"
  '';
in
{

  #无关内容省略
  
  nixpkgs.config.allowUnfree = true;
  services.xserver.videoDrivers = [ "nvidia" ];
  hardware.nvidia.prime = {
    offload.enable = true;
    intelBusId = "PCI:0:2:0";  # 此处的 BusId 需要通过命令 lspci | grep VGA 来查看
    nvidiaBusId = "PCI:1:0:0"; # 同上
  };
}
```
完成！