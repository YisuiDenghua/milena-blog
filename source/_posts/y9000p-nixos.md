---
title: 在联想拯救者 Y9000P 上安装 NixOS
categories: 
  - NixOS
  - 教程
#pic: https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/NixOS_logo.svg/1280px-NixOS_logo.svg.png
author: 
 - 一穂灯花
 - NixOS Chinese Community
date: 2022-05-06 09:15:59
tags: 
 - NixOS
 - Linux
 - 保姆级教程
---

<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>NixOS<br>
</h1>

<div class="info">

# 已知问题

扬声器无法使用，可使用 3.5mm 耳机或蓝牙耳机，或 HDMI 实现音频输出。



# LiveCD 

 由于 NixOS 官方 LiveCD 中使用的 5.15 内核较旧，且没有图形驱动，无法在拯救者 Y9000P 上显示出 GUI。因此，我们可以采用最小化（Minimal）安装镜像。



# 连接 WiFi

在最小化（Minimal）的安装镜像中，NetworkManager 不是可用的，所以必须手动配置网络。要配置 WiFi，首先运行 `sudo systemctl start wpa_supplicant` ，然后运行 `wpa_cli`。对于大多数家庭网络，需要收入如下命令：



```bash
> add_network
0
> set_network 0 ssid "我的网络名称"
OK
> set_network 0 psk "我的密码"
OK
> set_network 0 key_mgmt WPA-PSK
OK
> enable_network 0
OK

```

一旦成功连接，你会看到如下揭示

```
<3>CTRL-EVENT-CONNECTED - Connection to 32:85:ab:ef:24:5c completed [id=0 id_str=]
```

现在，可输入 `quit` 以离开 `wpa_cli`。

# 分区

预先在 Windows 中使用 `diskmgmt.msc` 缩小现有分区，为 NixOS 的安装留出空间。然后在 NixOS 的 livecd 中使用 `cfdisk`在空闲空间中创建一个约 500MB 的 EFI 分区，将其余空闲空间作为 NixOS 的根目录。



# 格式化

> 假设引导分区为 `/dev/nvme0n1p6/`，系统分区为 `/dev/nvme0n1p7`

```bash
mkfs.ext4 /dev/nvme0n1p7
mkfs.fat -F32 /dev/nvme0n1p6/
```



# 挂载

```bash
mount /dev/nvme0n1p7 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p6/ /mnt/boot
```



# 配置

使用 `nixos-generate-config` 来生成配置文件

```bash
nixos-generate-config --root /mnt
```



## 使用 5.19 内核

5.19 内核可支持拯救者 Y9000P 开启 165hz 屏幕刷新率。

首先需要将 Nix 发行渠道（Channel）改为不稳定版（Unstable）：

```bash
sudo nix-channel --add nix-channel --add https://mirrors.ustc.edu.cn/nix-channels/nixos-unstable nixos
sudo nix-channel --update
```



然后在 `/mnt/etc/nixos/configuration.nix` 中加入：

```nix
  boot.kernelPackages = pkgs.linuxPackages_testing;
```

目前，NixOS 不稳定版的的测试内核版本为 5.19rc5。



## 使用 NVIDIA PRIME

在 `configuration.nix` 中添加：

```
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

  nixpkgs.config.allowUnfree = true; # NVIDIA 驱动是非自由软件
  services.xserver.videoDrivers = [ "nvidia" ];
  hardware.nvidia.prime = {
    offload.enable = true;
    intelBusId = "PCI:0:2:0";  # 此处的 BusId 需要通过命令 lspci | grep VGA 来查看
    nvidiaBusId = "PCI:1:0:0"; # 同上
  };
  
  environment.systemPackages = with pkgs; [
    nvidia-offload # 安装 NVIDIA 支持软件
  ];
  
}
```



## 启用网络

```nix
networking.networkmanager.enable = true; 
```





# 安装 NixOS

到此为止，必要的配置就完成了。根据你的需求，可在进行其他配置后安装 NixOS。

```
sudo nixos-install --root /mnt --option substituters "https://mirror.sjtu.edu.cn/nix-channels/store"
```

