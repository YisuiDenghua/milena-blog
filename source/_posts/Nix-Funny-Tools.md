
---
title: 教程：合法但有病
categories: 
  - NixOS
  - Nix
  - 教程
author:
  - 一穗灯花
date: 2022-10-24 18:40:00
tags:
  - NixOS
  - Nix
---

<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>不用说你也知道这是啥<br>
</h1>

> 警告：这是个**生草**文章，我知道，但它写得太离谱了，甚至我们的客座研究员安德烈博士都对它作出了评价。我们快瞧瞧他说了些什么：

>*「哦，我的上帝，这真是棒呆了！我是说，我会让我的每一个学生都在他们的电脑上安装 NixOS 并将配置文件发送给我。什么？你不想这么做，哦，那么，我会用我那双打了蜡的皮鞋狠狠地踢你的屁股！我向上帝发誓，那感觉比吃了一杯煮烂的毛毛虫还要难受一百倍！」*——安德烈博士

好了伙计们，我想你们已经听了博士的话而去乖乖地安装一个 NixOS 了。那么，为了显得庄严一些，我们现在就要开始这个话题了！

# 使用 `configuration.nix` 设置密码

嘿！晚上好，我是一名黑客，正在偷看你电脑里的文件……哦不不不，不是的。真见鬼！我又说漏嘴了。我的意思是说，我是这里最精通 Nix 的人。现在由我来教给你，**使用 `configuration.nix` 来声明你的密码**：

我想你已经在安装 NixOS 的时候使用 `configuration.nix` 创建了一个用户了。不过，在这之后，你还需要给你的帐户设置一个密码，像这样：

```
# configuration.nix
{ ... }:
{
  users.users.你的名字 = {
    isNormalUser = true;
    extraGroups = [ "wheel" "networkmanager" ];
    password = "你的密码";
  };
}
```  

看好了，这是一种方式。像这样，**使用 `password` 来声明你的密码！**这样，我就可以看到你的密码了。

另外，你还可以**使用一个文件来声明你的密码**，只要把 `password` 换成 `passwordFile` ，然后将它后面的等号后面的密码字符串换成你的密码文件就好！

什么？你说，你不想让我看到你的密码？哦，天哪，这听起来多么令人扫兴。不过，在 NixOS 上，你是有这么做的权利的。来，使用 `hashedPassword` 来**设置一个加密的密码**。

> 提示：你可以使用 `mkpasswd -m sha-512` 来生成一个加密的密码

我敢打赌，你一定不情愿这么去做的。但万一有一天，你想用 `nixos-rebuild build-vm` 去测试你的配置的时候，你必然会想起我说的话的。嘿，我是说，如果你不在 `configuration.nix` 里设置密码，你是无法登录你的虚拟机的！

# 使用 `configuration.nix` 连接 Wifi

嘿！又是我，我是一个黑客……哦不是的，我是说，我并没有在偷看你电脑里的文件。相反，我是这里最懂 Nix 的人！相信我是没有错的。既然你已经使用了 Linux，我想你已经知道了怎么让它连网。但是，这里还有一个办法，可以**在 `configuration.nix` 里配置你的无线网络！**这样我就能方便地偷看你的网络……哦不，我是说，NixOS 可以使用 `wpa_supplicant` 来实现声明式的网络配置，像这样：

```
# configuration.nix
{ ... }:
{
    networking.wireless.enable = true;
    networking.wireless.networks = {
        "你的 Wifi 名称" = {
            psk = "你的密码";
        };
    };
}
```

什么？你觉得明文密码不安全？好吧，好吧，我答应你。多亏了 Nix，我们还可以**设置一个加密的密码**。

现在打开**终端**，在里面输入这些东西：

```
wpa_passphrase 名称 密码
```

然后，它会返回这样的东西：

```
network={
        ssid="你的 Wifi 名称"
        #psk="明文密码"
        psk=加密后的密码
}
```

然后，将我刚刚提到的那个 Nix 配置语句里的 `psk` 替换为 `pskRaw`，后面跟着你的加密后的密码

```
# configuration.nix
{ ... }:
{
    networking.wireless.enable = true;
    networking.wireless.networks = {
        "你的 Wifi 名称" = {
            pskRaw = "加密的密码";
        };
    };
}
```

或许你又想问了，这究竟有什么用呢？好吧，好吧。假如你有一个没有屏幕的树莓派，你可以在一个磁盘镜像中使用这个 `configuration.nix` 来构建一个系统，然后将此磁盘镜像刷入到树莓派中。这样，树莓派就会自动连接到你所设置的网络，使得你可以轻松地访问它，这听起来很酷！不是吗？

# 使用 `configuration.nix` 设置桌面壁纸

真是不巧！刚刚有一个黑客，他入侵了我的电脑并在修改了这篇文档！我不相信他会这么做。不过，看样子他已经替我写完前两部分了，那么，来让我们来到今天的最后一个话题：**使用 `configuration.nix` 设置桌面壁纸**。

但是，你知道的，这有些麻烦，要分两部分。不过不用担心，看在安德烈博士的份上，我们就将它分为两部分来详细解说。

1). 将你的图像放在 `$HOME/.background-image` 

2). 修改 `configuration.nix`

首先，如果你已经在 NixOS 上启用了「桌面」的话，我想你已经配置好了 X11 服务。总结起来无非这一点：

```
  services.xserver.enable = true;
```

这意味着，你的 X11 服务是启动了的。同样，X11 也可以用来**配置你的壁纸**。将以下内容写入到 `configuration.nix` ：

```
  services.xserver.desktopManager.wallpaper.mode = "fill";
```

当然，这里的 `fill` 也可以被替换成其他值。例如，`center` 意味着居中，`fill` 意味着填充，`max` 意味着最大化，`scale` 意味着拉伸，`tile` 意味着平铺。

或许你会好奇：这有什么用？好吧，好吧，虽然大多数桌面环境都提供了一个设置壁纸的菜单，可以随时随意地设置壁纸且不用重构 NixOS。但是，如果你想要 [创建一个 NixOS 的 LiveCD](https://github.com/nix-community/nixos-generators) 的话，这个设置将允许你使用自定义的桌面壁纸。听起来很酷！不是吗？

好了，今天就说这么多，来让我们听听安德烈博士有什么想说的。

> *「真是太酷了！我想，你最好现在就打开你的 NixOS 来实践一下，并且，使用你的配置创建一个 NixOS 的 LiveCD 并发送给我。」*

感谢你的阅读，好了，现在快去打开你的 NixOS 并试试这些有趣的功能吧！

