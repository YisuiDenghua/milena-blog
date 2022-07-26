---
title: 使用 Nix 表达式配置 Minecraft JE 服务器
categories: 
  - Minecraft
  - NixOS
  - 教程
author:
  - 一穗灯花
  - NixOS Community
date: 2022-07-19 19:06:00
tags:
  - Minecraft
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

> 警告：本篇内容针对 Minecraft Java Edition 的原版（Vanilla）服务端, 不适用于 Minecraft Bedrock Edition 以及使用 Fabric Mod 的服务端。有关 Fabric 服务端，参见 [Yinfeng 的 Nix Flake](https://github.com/linyinfeng/mc-config) 。

Minecraft 是一个关于破坏和放置方块的游戏。游戏一开始玩家的主要目的是搭建各种结构使自己免遭夜晚出没的怪物的攻击并生存下来，但随着游戏的进行，玩家们可以合作创造出一些不可思议的、富有想象力的东西。 


# 部署服务器

由于 Minecraft 服务端是非自由软件，你需要先在 `configuration.nix` 中允许非自由软件：

```nix
  nixpkgs.config.allowUnfree = true;
```

Minecraft 服务器可作为一个 NixOS 模块被启用。

若要启用 Minecraft 服务器，将下列内容加入 `configuration.nix` ：

```nix
  services.minecraft-server.enable = true;
```

同时，可使用 `services.minecraft-server.dataDir` 描述服务器数据存储的目录，默认为 `var/lib/minecraft` 。

可使用 `services.minecraft-server.package` 描述服务端的版本。默认情况下，NixOS 将使用最新版本的官方 Minecraft 服务端。

例如，如果要启用 Minecraft 1.12.2 的服务端，将下列内容加入 `configuration.nix` ：
 
```nix
  services.minecraft-server.package = pkgs.minecraft-server_1_12_2;
```

# 声明式的配置

NixOS 允许使用 Nix 表达式语言来对 Minecraft 服务器进行声明式的配置。若要使用 Nix 来配置 Minecraft 服务器，将下列内容加入  `configuration.nix` ：

```
  services.minecraft-server.declarative = true;
```

# 同意许可协议

要启用 Minecraft 服务器，你需要同意 [Mojang 的用户许可协议](https://account.mojang.com/documents/minecraft_eula)。如果你同意这些协议，将下列内容加入 `configuration.nix` :

```nix
  services.minecraft-server.eula = true;
```

# 配置服务器

在 NixOS 上，Minecraft 服务器的配置由 `services.minecraft-server.serverProperties` 模块来描述。

例如：

```nix
  services.minecraft-server.serverProperties = {
    server-port = 43000;
    difficulty = 3;
    gamemode = 1;
    motd = "NixOS Minecraft server!";
    max-players = 5;
    online-mode = false;
  };
  
```

可以在 [Minecraft Wiki](https://minecraft.fandom.com/wiki/Server.properties#Java_Edition_3) 里查看 Server Properties 的可用配置。


# 白名单

若要开启白名单制度，需在 `services.minecraft-server.serverProperties` 模块中将 `white-list` 子对象设置为 `true` ，并将下列内容加入 `configuration.nix` :

```nix
  services.minecraft-server.whitelist = {
    玩家甲 = "uuid-a";
    玩家乙 = "uuid-b";
  };
```

# 防火墙

若服务器开启了防火墙保护，可在 `configuration.nix` 中添加如下内容以放行 Minecraft 端口：

```nix
  services.minecraft-server.openFirewall = true;
```

# JVM 参数
在 NixOS 上，可使用 `services.minecraft-server.jvmOpts` 模块配置 Minecraft 服务器运行所使用的 JVM 参数。

例如：

```nix
  services.minecraft-server.jvmOpts = "-Xmx2048M -Xms2048M";
```
