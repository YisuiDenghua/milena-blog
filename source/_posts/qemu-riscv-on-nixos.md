---
title: 在 NixOS 上使用 QEMU 运行 RISC-V 64 版本的 openEuler
categories: 
  - NixOS
  - 教程
#pic: https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/NixOS_logo.svg/1280px-NixOS_logo.svg.png
author: 
 - 一穂灯花
 - NixOS Chinese Community
date: 2022-08-08 07:46:22
tags: 
 - NixOS
 - Linux
---

<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>NixOS<br>
</h1>

<div class="info">


# 1. 编译和安装 QEMU

在 `configuration.nix` 中添加如下内容：

```
  boot.binfmt.emulatedSystems = [
    "riscv64-linux"
  ];

  boot.extraModprobeConfig = "options kvm_intel nested=1"; # 启用 KVM
  # 如果使用 AMD 的芯片，将上行代码中的 kvm_intel 改为 kvm_amd

  virtualisation.libvirtd.enable = true;
  programs.dconf.enable = true;
```

> 注意：编译 QEMU 可能会占用大量时间、CPU 和系统内存。

# 2. 运行 RISC-V 64 版的 openEuler

## 2.1 下载 openEuler

从 (https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/testing/20220622/v0.2/QEMU/) 下载 fw_payload_oe_qemuvirt.elf, openeuler-qemu.raw.tar.zst, start_vm.sh 。

新建一个文件夹，并将刚刚下载的文件移动到该文件夹下。

在终端里使用 `cd` 命令进入该文件夹。

## 2.2 修改脚本

编辑 `start_vm.sh` 脚本，将其开头的 `#!/bin/bash` 改为 `#!/usr/bin/env bash` 以适应 NixOS 的根目录结构

使用 `chmod` 为脚本赋予执行权限

```
chmod +x start_vm.sh
```

# 2.3 运行 openEuler

在终端输入下列命令，稍后 openEuler 的 tty 界面会出现在当前的终端窗口中。

```
./start_vm.sh
```

默认的用户名是 root, 密码是 openEuler12#$ 。建议在启动系统之后通过 `passwd root` 更改密码。