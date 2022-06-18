---
title: 一个配置文件就能在各个电脑装一模一样的系统和软件？——保姆级教你轻松掌握NixOS
author: 
 - Lanta
 - 一穂灯花
 - Chi_Tang
 - NixOS Chinese Community
date: 2022-05-06 09:15:59
tags: 
 - NixOS
 - Linux
 - 保姆级教程
categories: 教程

---

<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>NixOS<br>
</h1>

<div class="info">


>非常感谢[一穂灯花](https://github.com/YisuiDenghua)在我安装NixOS中提供的帮助，以完成我对NixOS的学习和本文章的创作

</div>


# NixOS 适合我吗？

>「如果你不需要 Ubuntu 那种的开箱即用的，觉得 Arch 那样的自己动手可以接受，喜欢 Gentoo 的可配置性，又不想自己完全编译整个世界，并且觉得所有系统配置全都由统一格式写出来的可复现性确实很棒，并且觉得那帮人为了 portable 和轻量级，自己做了个 Nix 语言可以接受的话，NixOS 可能就是专门为你设计的。」—— [Dramforever](https://github.com/dramforever)，NixOS 中文社区管理员之一

# 前排提示

由于 NixOS 系统的配置，操作方式和传统的 Linux 发行版有很大不同，建议新手在先在虚拟机中试用 NixOS，等到熟悉了之后，将 configuration.nix 等配置文件导出，而后能够在宿主机上使用相同的配置复现出一个一样的 NixOS。

在安装的时候，建议新人阅读 NixOS 自带的手册 NixOS Manual，以及在 [search.nixos.org](search.nixos.org) 上查询软件包名（Packages）和配置语句（Options）



>永远不要想着用常规 Linux 方式在 NixOS 上安装软件。因为 NixOS 不遵守 fhs，也就是说，它的根目录结构和常规的 Linux 是不一样的。不信你看看，NixOS 的 /bin 下什么都没有。—— [一穂灯花](https://github.com/YisuiDenghua)

# 认识NixOS

NixOS 是一个基于 Nix 包管理器和 Nix 构建系统的 Linux 发行版。它支持可重现和声明性的系统配置管理以及事务回滚，它也支持命令式的软件包和用户管理。在 NixOS 中，发行版的所有组件——包括内核，安装的包和系统配置文件都是由 Nix 表达式构建的。

## 软件包的安装方式

NixOS 非常不同，如果你想安装软件，有多种安装方式，这里举三个例子

1. 声明式的软件包管理（编辑 NixOS 的配置文件`/etc/nixos/configuration.nix`）

<div class="info">


>安装在`configuration.nix`的软件就是构建在系统里面了，而且因为在配置文件里面，下次你要安装NixOS到其他电脑，只要用这个配置文件（可能需要稍微修改一下）就可以轻松安装NixOS顺便安装你需要的软件了
>
>比如说我需要装git，把 git 写进`configuration.nix`
>
> ```nix
> environment.systemPackages = with pkgs; [ 
> git
> ];  
> ```
>
>然后 `nixos-rebuild boot`重启就可以安装完成
>
>缺点是，需要重启（因为重构了部分系统文件），不过重构的时间并不多，基本上几分钟就可以了

</div>

<div class="warning">


>需要注意的是，有一些软件的开源协议是**不自由**的，所以NixOS会阻止你的安装操作
>
>要解决这个问题也很简单，只要在`configuration.nix`中允许即可，在里面加入`nixpkgs.config.allowUnfree = true;`
>
>例如：
>
>![](https://pic.lanta.cyou/img/2022-05-06_09-45.png)

</div>

2.`nix-shell`临时使用

<div class="info">


>`nix-shell`的安装是临时使用，比如我要临时用一下git，那就`nix-shell -p git`
>
>随后，你会进入一个 nix-shell，在这个 nix-shell 当中，你可以使用 git。当你用 `exit`退出 nix-shell 之后，git 就不再可用。

</div>

3. 命令式的软件包管理（`nix-env -iA nixos.<包名>`）

<div class="warning">

> 不推荐使用这种方式安装软件

</div>

<div class="info">


> 这种不需要 sudo，也不需要重构，它是把软件安装在用户的目录下
>
> 缺点是，这种方式安装的软件不能被 configuration.nix 记录，也不能被其他用户使用
>
> ——[一穂灯花](https://github.com/YisuiDenghua)

</div>

## NixOS 的配置是可复现的

NixOS跟其他Linux发行版最大的不同就是，NixOS 的配置是可复现的。几乎所有的系统设置都在`configuration.nix`

例如，添加用户、安装的软件、字体、网络、系统引导、桌面环境，全部包含在了这么一个文件里面。而不是像其他发行版那样，配置文件散落在系统各处。

也就是说，只要你在`configuration.nix`配置好之后，下次你需要安装到其他电脑只要再用这个配置文件然后`nixos-install`就可以得到一模一样的NixOS。

同理，如果你去Github下载别人的`configuration.nix`，你可以得到跟他们一模一样的NixOS



>  笔者的[`configuration.nix`](https://github.com/RealLanta/nixos-config)

# 准备操作

首先你需要确保你的网络畅通，NixOS是**在线安装**。也就是说，安装的过程需要保持网络连接。

## 下载LiveCD

你可以前往[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/nixos-images/)中下载NixOS的Live CD

![](https://pic.lanta.cyou/img/2022-05-06_09-47.png)

<div class="warning">


>请尽量下载较新的版本

</div>

里面有很多个镜像，这里做一下解释

![](https://pic.lanta.cyou/img/2022-05-06_09-50.png)

## 刻录到U盘

你可以使用以下几种方式来将NixOS镜像刻录到U盘中

<div class="danger">



> 刻录到U盘会导致数据丢失，请在操作之前备份好你的U盘数据

</div>

<div class="warning">

>警告：Ventoy 的硬件支持比较有限。在使用 Ventoy 前，需先阅读其官网的文档，以了解硬件支持情况

</div>

| 名称                                          | 使用方式                                        | 备注               |
| --------------------------------------------- | ----------------------------------------------- | ------------------ |
| [Ventoy](https://ventoy.net/)                 | 将Ventoy安装到U盘后直接把ISO文件复制到U盘中即可 | 一次安装，一盘多用 |
| [Rufus](https://rufus.ie/)                    | 选择ISO文件然后选择U盘直接刻录即可              | 注意选择引导方式   |
| [balenaEtcher](https://www.balena.io/etcher/) | 选择ISO文件然后选择U盘点击Flash开始刻录         | 最简单的方法       |

## 确定分区（必要）

在纯文字的环境中你可能**很难分辨**分区，如果**操作失误**（删错分区）~~则删库跑路~~

所以我建议你**事先**在Windows环境中**确定**要安装的系统分区，以确保不会出现失误导致的~~数据灰飞烟灭~~


## 引导设置（UEFI用户）

进入BIOS中，如果你是有UEFI的电脑那么你应该能看到带有**CSM**字样的选项，将他**Disable**掉

还有一些电脑带有**Secure Boot（安全启动）**，找到**Secure Boot**的字样**Disable**即可

## 重启开始安装

确保您的U盘已经写入了镜像、分区已经确定好没问题之后就可以重启进入LiveCD

# LiveCD

## 进入NixOS的root

进入终端后你会发现NixOS默认登录的不是root账户，首先你需要登录到root账户才能继续

打开终端，输入`sudo -i`即可进入root账户

## 确保网络畅通

前面提到，NixOS是在线安装方式，所以你必须要确保LiveCD中的系统已经连接到了网络

### 有线网络

一般来说如果你使用了带桌面环境的ISO就已经可以自动联网

不过发现网络出现玄学问题，你可以执行`systemctl restart dhcpcd`来重启DHCP服务让Live CD重新获取IP

### WiFi连接

提供了 GUI 的 LiveCD 中自带 NetworkManager，你可以一步一步让Live CD连接到WiFi

```bash
[root@nixos ~]# nmcli device wifi connect <WIFI名> password <密码>
```

然而，在最小化的纯命令行安装镜像中，NetworkManager 不是可用的，所以必须手动配置网络。要配置 WiFi，首先使用 `sudo systemctl start wpa_supplicant`，然后运行 `wpa_cli`。对于大多数家庭网络，你需要像这样输入命令：

```shell
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

对于企业网络，例如 *eduroam*，应该这样做：

```shell
> add_network
0
> set_network 0 ssid "eduroam"
OK
> set_network 0 identity "myname@example.com"
OK
> set_network 0 password "mypassword"
OK
> set_network 0 key_mgmt WPA-EAP
OK
> enable_network 0
OK
```

一旦成功连接，你会看到这样的提示：

```shell
<3>CTRL-EVENT-CONNECTED - Connection to 32:85:ab:ef:24:5c completed [id=0 id_str=]
```

现在，你可以通过输入 `quit`来离开`wpa_cli`。



> 提示：如果你想要在另一台机器上远程安装，你可以在联网之后，使用`systemctl start sshd` 激活 OpenSSH 守护程序 。另外，你还必须使用 `passwd` 为 root 用户或 nixos （livecd 的默认用户）设置密码。

## 分区

先通过`lsblk`或`fdisk -l`的方式确定硬盘，出现`/dev/某某某`的就是这个硬盘的设备名，记住硬盘的设备名，等会要用到

### 分区的工具

LiveCD中有`parted`、`fdisk`、`cfdisk`等工具，前两个比较麻烦，`cfdisk`拥有半图形界面，更适合新手使用

如果使用带有 GUI 的 livecd，则 `gparted` 也是可用的。这里以 cfdisk 为例：

首先我们要打开cfdisk

<div class="info">



> 设sdX为你的硬盘号

</div>

```bash
cfdisk /dev/sdX # 后面这个就是硬盘设备名
```

然后就会出现一个界面，这里以一个中文化的界面作为对照

![](https://pic.lanta.cyou/img/20220326033131.png)

### 分区格式

<div class="info">



>  用户资料分区是可选的，可以将用户资料存放在系统盘中

</div>

#### MBR（传统引导）推荐分区格式：

| 用途                 | 文件系统 | 分区推荐大小 | 挂载点 | 文章中的分区号                   |
| -------------------- | -------- | ------------ | ------ | -------------------------------- |
| 系统分区             | ext4     | 50GiB        | /      | /dev/sda2                        |
| 用户资料分区（可选） | ext4     | 50GiB        | /home  | （本文章中没有设置用户资料分区） |

#### GPT（UEFI引导）推荐分区格式：

<div class="danger">



> 如果你之前已经在硬盘中另外装了其他使用UEFI引导的系统（比如Windows）请务必不要把引导分区删掉重建，否则会导致原来的系统无法进入！

</div>

<div class="danger">



> 因为某些机制的原因，请记得在`cfdisk`中把引导分区的“类型”(Type)改为"EFI System"

</div>

| 用途                 | 文件系统 | 分区推荐大小 | 挂载点 | 文章中的分区号                   |
| -------------------- | -------- | ------------ | ------ | -------------------------------- |
| 引导分区             | FAT32    | 300MiB       | /efi   | /dev/sda1                        |
| 系统分区             | ext4     | 50GiB        | /      | /dev/sda2                        |
| 用户资料分区（可选） | ext4     | 50GiB        | /home  | （本文章中没有设置用户资料分区） |

> 提示：如果你选择将 EFI 引导分区挂载到不是 `/boot` 的挂载点，例如 `/efi`，你还需要在接下来编辑 `configuration.nix` 的时候写入
>
> ```
> boot.loader.efi.efiSysMountPoint = "/efi";
> 
> ```
>
> 其中，`/efi`可以替换为你所使用的挂载点

## 格式化分区

<div class="danger">



> 如果你之前已经在硬盘中另外装了其他使用UEFI引导的系统（比如Windows）请务必不要把引导分区格式化，否则会导致原来的系统无法进入！

</div>

<div class="info">



> 设引导分区为 /dev/sda1，系统分区为 /dev/sda2

</div>

```bash
mkfs.ext4 /dev/sda2 # 格式化 /dev/sda2 为 ext4 文件系统（系统分区或用户资料分区）
mkfs.vfat /dev/sda1 # 格式化 /dev/sda1 为 FAT32 文件系统（引导分区）
```

## 挂载分区

### GPT

<div class="info">



> 设引导分区为 /dev/sda1，系统分区为 /dev/sda2，用户资料分区为/dev/sda3

</div>

```bash
mount /dev/sda2 /mnt # 挂载系统分区到 /mnt( / 分区)
mkdir /mnt/boot /mnt/home # 新建 /mnt/boot 文件夹和 /mnt/home 文件夹
mount /dev/sda1 /mnt/boot # 挂载引导分区到 /mnt/boot
mount /dev/sda3 /mnt/home # 挂载用户资料分区到 /mnt/home （ /home 分区，在有用户资料分区的情况下才执行）
```

### MBR

```bash
mount /dev/sda1 /mnt  # 挂载系统分区到 /mnt( / 分区)
mkdir /mnt/home # 创建 /mnt/home 文件夹
mount /dev/sda2 /mnt/home # 挂载用户资料分区到 /mnt/home( /home 分区，在有用户资料分区的情况下才执行)
```

（以上文段参考了[Chi_Tang的博客](https://chitang.tech/posts/arch-guide/#%E6%8C%82%E8%BD%BD)并进行修改）

## 安装本体到硬盘

### 使用中国的 nix-channel

```shell
nix-channel --add https://mirrors.ustc.edu.cn/nix-channels/nixos-unstable nixos
nix-channel --update
```

### 生成配置文件

```bash
 # nixos-generate-config --root /mnt
```

该命令会生成配置文件到`/mnt/etc/nixos/`目录中，里面包括了硬件配置文件(`hardware-configuration.nix`)和`configuration.nix`

如果你有自己配置好的`configuration.nix`只要替换掉即可

> 提示：在修改 NixOS 配置文件的时候要注意缩进及分号 `;` 的使用。`configuration.nix` 当中已经存在一些默认配置好的语句，可以参考它们的格式。

### 更改配置文件

> 「古人云：『知其然，知其所以然。（宋・朱熹《建宁府建阳县长滩社仓记》）』意思是说，在知道事物的表面现象的同时，更应去探明事物的本质及其产生的原因。同样的道理可以应用在 Nix 上。在配置 NixOS 的时候，总抄作业可不是好习惯。抄袭其他人的配置文件能够满足用户一时的使用，但此时的用户仍然没有了解 Nix 的设置方法，当用户真正有自己需求的时候，仍然不知该怎么做。相比起抄袭其他人配置文件，我更希望用户阅读 NixOS 自带的手册 NixOS Manual，以及在 [search.nixos.org](search.nixos.org) 上查询软件包名（Packages）和配置语句（Options）。这样，用户能真正学到 NixOS 独特的声明式配置方式的精髓。」 ——[一穗灯花](github.con/YisuiDenghua)



使用vim开始编辑

```bash
vim /mnt/etc/nixos/configuration.nix
```

#### 允许安装不自由的软件

```nix
  nixpkgs.config.allowUnfree = true;
```

![](https://pic.lanta.cyou/img/2022-05-06_10-15.png)

#### 安装NotoFonts字体以显示中文和其他特殊字符

```nix
  fonts.fonts = with pkgs; [
    noto-fonts
    noto-fonts-cjk
    noto-fonts-emoji
    meslo-lgs-nf
  ];
```



![](https://pic.lanta.cyou/img/2022-05-06_10-16.png)

#### 设置时区

```nix
  time.timeZone = "Asia/Shanghai";
```



![](https://pic.lanta.cyou/img/2022-05-06_10-16_1.png)

#### 使用NetworkManager联网

```nix
  networking.networkmanager.enable = true;
```



![](https://pic.lanta.cyou/img/20220506104638.png)

#### 设置中文

```nix
  i18n.defaultLocale = "zh_CN.UTF-8";
  i18n.inputMethod.enabled = "fcitx5";
  i18n.inputMethod.fcitx5.addons = with pkgs; [ fcitx5-rime ];

```

> 提示：这里使用 `fcitx5-rime `作为示例。用户可以将其替换为 `fcitx5-chinese-addons` 等其他输入法。

![](https://pic.lanta.cyou/img/2022-05-06_10-19.png)

#### 设置桌面环境

如果你使用的是Plasma，那么生成配置文件的时候会自动帮你设置好Plasma，Gnome用户同理。这里以 KDE Plasma 和 GNU GNOME 为例：

<div class="warning">

> 警告：在 NixOS 上，无法同时启用 GNOME 和 KDE Plasma。

</div>

```nix
  # 如果要使用 GNOME：
  services.xserver.enable = true;
  services.xserver.displayManager.gdm.enable = true;
  services.xserver.desktopManager.gnome.enable = true;
```

```nix
  # 如果要使用 KDE Plasma
  services.xserver.enable = true;
  services.xserver.displayManager.sddm.enable = true;
  services.xserver.desktopManager.plasma5.enable = true;

```

> 提示：如果想要启用 Plasma 的 Qt 缩放，你还需要添加一条语句：
>
> ```   services.xserver.desktopManager.plasma5.useQtScaling = **true**;
> services.xserver.desktopManager.plasma5.useQtScaling = true;
> ```

![](https://pic.lanta.cyou/img/2022-05-06_10-20.png)

#### 设置普通用户

```nix
  users.users.<用户名> = {
    isNormalUser = true;
    shell = pkgs.zsh;
    extraGroups = [ "wheel" ]; # Enable ‘sudo’ for the user.
  };

```

> 提示，如果你的用户要使用图形界面，你还需要将 `"video"` 加入到  `extraGroups = [  ];  ` 当中；如果你的用户要使用 `docker`，你还需要将`"docker"`加入语句中。如果你不想给用户使用 `sudo` 的权利，将  `"wheel" ` 从上述语句中移除。

![](https://pic.lanta.cyou/img/2022-05-06_10-22.png)

#### 设置安装的软件

![](https://pic.lanta.cyou/img/2022-05-06_10-25.png)

#### 开启SSH（可选）

![](https://pic.lanta.cyou/img/2022-05-06_10-26.png)



### 提示：用户的特殊需求

对于某些有特殊需求的用户，需要在安装之前按需修改配置文件。

#### 使用 NUR

```
  nixpkgs.config.packageOverrides = pkgs: {
    nur = import (builtins.fetchTarball "https://github.com/nix-community/NUR/archive/master.tar.gz") {
      inherit pkgs;
    };
  };  
    
```

而后，可以在 https://nur.nix-community.org/ 上搜索 NUR 软件包，例如微信。然后，可以使用声明式的方式安装：

![](https://pic.lanta.cyou/img/photo_2022-05-06_13-49-27.jpg)

```nix
  environment.systemPackages = with pkgs; [
    nur.repos.xddxdd.wechat-uos-bin
   ];
```

> 提示：由于 NUR 使用 [GitHub](github.com) 来托管软件包代码，而 GitHub 在国内的某些地区连接不佳，你可能还需要在安装之前配置代理。
>
> 可以通过 `nix-env -iA nixos.v2ray` 在安装 LiveCD 里临时安装 `v2ray`。对于带有 GUI 的 LiveCD，可使用 `nix-env -iA nixos.qv2ray`。在使用 `qv2ray` 的时候，需参照 https://qv2ray.net/lang/zh/getting-started/step2.html 配置其核心文件。

#### 使用 AMD 显卡

```
hardware.opengl.extraPackages = [
  rocm-opencl-icd
  pkgs.amdvlk
];
```

#### 使用 Intel Gen8 或更新的显卡

```
hardware.opengl.extraPackages = [
  intel-compute-runtime
];
```

#### 使用 NVIDIA 显卡

<div class="warning">

> 警告：如果你正在为 Linux 选购硬件，除非特殊需要，请避免使用由 NVIDIA，Realtek，Broadcom 以及 Apple Inc 生产的设备。在这些公司所生产的硬件设备上部署 Linux 可能会非常棘手。

</div>

```nix
  services.xserver.videoDrivers = [ "nvidia" ];
```

<div class="info">

> 提示：由于 NVIDIA 的驱动是非自由软件，你同时还需要在配置文件中使用 `  nixpkgs.config.allowUnfree = true;`启用非自由软件

</div>

#### 使用数位板

```nix
  services.xserver.wacom.enable = true;
  services.xserver.digimend.enable = true;
  hardware.opentabletdriver.enable = true;
  hardware.opentabletdriver.daemon.enable = true;
```

#### 使用 Steam

```
  programs.steam.enable = true;
```

#### 使用 AppImage

安装软件包 `appimage-run`

#### 使用为常规 Linux 而编译的二进行程序

安装软件包 `steam-run`

#### 在 NixOS 中作为宿主时使用 VirtualBox

```nix
  virtualisation.virtualbox.host.enable = true;
  users.extraGroups.vboxusers.members = [ "<用户名>" ];
```

#### 在 NixOS 中作为宿主时使用 KVM

```nix
  virtualisation.libvirtd.qemu.package = pkgs.qemu_kvm;
  virtualisation.libvirtd.enable = true;
```

#### 使用 Flatpak

<div class="warning">

>如果你这 Flatpak 安装了有中文的软件，请在 `~/.local/share/fonts/` 目录里放几个中文字体，否则中文无法正常显示

</div>

```nix
  services.flatpak.enable = true;
```

> 提示：如果你使用非 GNOME 的桌面环境，你还需要配置
>
> ```nix
> xdg.portal.extraPortals = [ pkgs.xdg-desktop-portal-gtk ];
> ```

在启用 flatpak 之后，需要导入 flathub 源：

```shell
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak update
```
#### 开启支持 NTFS 读写

<div class="warning">

> 如果你有使用 NTFS 格式的硬盘，在 `configuration.nix` 中加入以下字段来提供 NTFS 的支持

```nix
boot.supportedFilesystems = [ "ntfs" ];
```

</div>

#### 使用 Docker

```nix
  virtualisation.docker.enable = true;
```

此外，你还需要将用户加入 "docker" 用户组

#### 使用其他 Linux 内核

`Packages` 选项来指定 NixOS 使用的内核。例如：使用 `linux-zen`

```nix
  boot.kernelPackages = pkgs.linuxPackages_zen;
```

#### 使用中国的 Nix 二进制源

```nix
  nix.settings.substituters = [ "https://mirrors.ustc.edu.cn/nix-channels/store" ];
```
<div class="warning">

> 注意
>
> 在NixOS 21.11中，不存在 `nix.settings` 段
>
> 所以如果你安装的是NixOS 21.11，要改成下面这样：
>
> `nix.binaryCaches = [ "https://mirrors.ustc.edu.cn/nix-channels/store" ];`

</div>

### 安装 NixOS

```bash
nixos-install --root /mnt
```

## 重启

<div class="success">

> 到这里，我们的LiveCD的安装步骤就完成了，曙光就在眼前了！接下来我们重启就可以进入你配置好的NixOS

</div>

