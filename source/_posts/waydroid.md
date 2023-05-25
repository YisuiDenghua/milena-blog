---
title: 教程：Waydroid
categories: 
  - NixOS
  - 教程
author: 
 - 一穂灯花
date: 2023-3-13 02:28:00
tags: 
 - NixOS
 - Linux
---

<h1 align="center">
  <img src="https://flatpak.org/img/distro/nixos-9dbc38e1.svg" width="200">
  <br>NixOS<br>
</h1>

## Waydroid
Waydroid 是一个基于 LXC 容器的工具，可以用于在常规 GNU/Linux 上运行 Android。它可以直接在 Wayland 桌面（如 GNOME，KDE Plasma 等桌面环境的 Wayland 会话）上运行，也可以通过 Weston 窗口管理器在 X11 桌面上运行。

### 1. 准备

#### 1.1 内核
从 Linux 5.18 内核开始，ashmem 从内核中被移除。如果使用原版内核，编辑 `/var/lib/waydroid/waydroid_base.prop` 并在其中加入
```
sys.use_memfd=true
```

或者，使用 linux zen 内核，在 `configuration.nix` 中加入如下内容：
```
  boot.kernelPackages = pkgs.linuxPackages_zen;

```

#### 1.2 CPU
Android 仅支持特定的 CPU 架构（ x86_64, x86, arm64-v8a armeabi-v7a）。可使用 `cat /proc/cpuinfo` 查看当前的 CPU 结构

#### 1.3 GPU
Waydroid 使用 Android 的 mesa 集成来实现穿透（passthrough），因此它在支持大多数移动端的 ARM/ARM64 SOC。 在 x86 的 PC 上，Waydroid 支持 Intel 与 AMD 的 GPU。如果你使用 NVIDIA GPU 或者是虚拟机，建议使用软件渲染。如果你不使用 NVIDIA 显卡或虚拟机，可跳过本文的 1.4 部分

#### 1.4 软件渲染（可选）
编辑 `/var/lib/waydroid/waydroid.cfg` 文件，并在其中加入如下内容

```
ro.hardware.gralloc=default
ro.hardware.egl=swiftshader
```

然后使用 `sudo waydroid upgrade -o` 命令应用更改。

#### 1.4 桌面环境
Waydroid 只能在 Wayland 会话中使用，很多常见的桌面环境（如 GNOME，KDE Plasma）已经支持 Wayland。GNOME 已经默认使用 Wayland 会话。若要默认使用 KDE Plasma 的 Wayland 会话，将如下内容加入到 `configuration.nix` ：

```
  services.xserver.displayManager.defaultSession = "plasmawayland";
```

如果你使用的是 X11 的桌面环境，则可使用 Weston 窗口管理器来运行 Waydroid。安装 `weston` 软件包。

### 2. 安装
> 警告：如果在运行 `nixos-generate-config` 之前就在系统上安装 WayDroid 会导致创建不必要的 fstab 条目，这些条目可能会干扰系统功能。

将以下内容添加到 `configuration.nix` ：

```
{ pkgs, ... }:
{
  virtualisation = {
    waydroid.enable = true;
    lxd.enable = true;
  };
}
```

添加之后，使用 `sudo nixos-rebuild switch` 应用更改。在这之后，使用

```
sudo waydroid init
``` 
命令来初始化一个 Waydroid。这个过程需要连接到 Sourceforge。

> 提示：如果你想要使用 GAPPS，则运行 `sudo waydroid init -s GAPPS -f` 命令。

### 使用

>提示：如果你使用 X11 桌面，你需要先运行 `weston` 命令来打开一个 Weston 窗口，然后在 Weston 窗口中用鼠标单击左上角终端图标，之后在 Weston 终端中执行如下命令。

使用以下命令
```
# 在第一次安装之后，需要开启 Waydroid LXC 容器
sudo systemctl start waydroid-container

waydroid session start
```

基本命令

```
# 启动 Android 完整桌面
waydroid session start
# 列出安装的 Android 应用
waydroid app list
# 启动 Android 应用
waydroid app launch <应用名称>
# 安装 APK
waydroid app install </path/to/app.apk>
# 进入 LXC Shell
sudo waydroid shell
# 启用多窗口模式
waydroid prop set persist.waydroid.multi_windows true
# 更新系统
sudo waydroid upgrade
```

### 网络

> 警告：笔者接触网络较晚，对网络及其实现原理不是很了解。以下内容可能有误。

> 如有错误，还请到 GitHub 上提出 Issue 或者 PR

Waydroid 的网络理应开箱即用。如果你的 Waydroid 没有网络，则需要在内核中开启数据包转发（Packet forwarding）功能，并允许以下规则通过防火墙。在 `configuration.nix` 中添加如下内容：

```
  # 开启数据包转发功能
  boot.kernel.sysctl = {
    "net.ipv4.ip_forward" = 1;
    "net.ipv6.conf.default.forwarding" = 1;
    "net.ipv6.conf.all.forwarding" = 1;
  };
  
  # 防火墙规则
  networking = {
    firewall = {
      enable = true; # 开启防火墙
      allowedUDPPorts = [ 67 53 ]; # 开启相关端口 
    };
  };
```




### 截图

图一：Waydroid 多窗口模式
![](/images/waydroid.png)

图二：Waydroid 的 Full UI 模式
![](/images/full_ui.png)

图三：在 Weston 中运行的 Waydroid Full UI 模式，Weston 可运行在 X11 的桌面上。
![](/images/waydroid-weston.png)




