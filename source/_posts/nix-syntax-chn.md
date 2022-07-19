---
title: NixOS 配置文件句法 - 第一部分
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

> 提示：本文除附加项目外的内容译自 *NixOS Manual Part II. Chapter 6*

NixOS 的配置文件 `/etc/nixos/configuration.nix` 是一个使用 Nix 包管理器的纯函数式语言编写的 Nix 表达式。这个文件中的表达式定义了整个 NixOS 系统，包括安装的软件包和系统配置。

# NixOS 配置文件
一个 NixOS 配置文件看起来通常是这样的
```Nix
{ config, pkgs, ... }:

{ 定义选项
}
```
其中第一行 `{ config, pkgs, ... }:` 表示在这个配置文件里使用了 `configs` 和 `pkgs` 参数。（参见 *NixOS Manual Chapter 66, Writing NixOS Moduls*）。表达式返回了一组选项的集合 `{ ... }`

NixOS 设置语句通常是 `选项 = 值` 这样的结构，这也是 Nix 语言的基本语法。

> 提示：你可以在 search.nixos.org 的 Options 条目下搜索选项名及其可用的值。

示例：
```Nix
{ config, pkgs, ... }:

{ services.httpd.enable = true;
  services.httpd.adminAddr = "yisui@denghua.org";
  services.httpd.virtualHosts.localhost.documentRoot = "/webroot";
}
```
上面的表达式使用三条选项（Option）定义了一个配置，以启用 Apache HTTP 服务器，并将 `/webroot` 作为文件根目录。

这个选项的集合可以嵌套，事实上，选项名称之间的点 `.` 是定义包含另一个集合的集合的简写。例如， `services.httpd.enable` 定义了一个名为 `services` 的集合，代表 NixOS 上可启用的系统服务，`services` 里包含了名为 `httpd` 的集合，该集合又包括了一个名为 `enable` 的子对象，我们给它赋值为 `true` 。这就意味着，上述例子中的表达式也可以写成：

```Nix
{ config, pkgs, ... }:

{ services = {
    httpd = {
      enable = true;
      adminAddr = "yisui@denghua.org";
      virtualHosts = {
        localhost = {
          documentRoot = "/webroot";
        };
      };
    };
  };
}
```

当你有很多条使用同一个前缀（例如 `services.httpd` ）的选项时，这种写法会更加便利。

NixOS 会检查你的选项是否正确。比如，如果你尝试定义一个不存在的选项， `nixos-rebuild` 会给出如下报错信息

```
The option 'services.httpd.enable' defined in '/etc/nixos/configuration.nix' does not exist.
```

> 提示：上述报错信息翻译成中文即为
> 
>  ` 定义于 '/etc/nixos/configuration.nix' 的选项 services.httpd.enable 不存在。`

与此同时，选项的赋值需要有一个正确的类型。比如， `services.httpd.enable` 必须是一个布尔值（ `true` 或 `false` ）。如果尝试去赋一个其他类型的值，则会遇到如下错误：

```
The option value 'services.httpd.enable' in '/etc/nixos/configuration.nix' is not a boolean.
```
> 提示：上述报错信息翻译成中文即为
> 
>  `  '/etc/nixos/configuration.nix' 中的选项 services.httpd.enable 的值不是一个布尔值。`

在完成更改之后，使用 `nixos-rebuild switch` 应用之。

# 附加项目：`nixos-rebuild` 命令的使用

`nixos-rebuild` 是一个 NixOS 命令，用于应用用户在系统配置文件中做出的更改。亦可用于一系列有关管理 NixOS 系统状态的其他事务。

当你对 NixOS 系统做出更改之后，你需要使用 `nixos-rebuild` 将它们应用。此时，NixOS 会根据配置文件进行重构。然而 `nixos-rebuild` 的运行需要子命令。不同的子命令可指定其完成不同任务。

## `nixos-rebuild` 的子命令

`nixos-build switch` 会重构你的系统，同时立刻应用新配置，并将它用作你的默认启动项。

`nixos-rebuild boot` 会重构你的系统，并将新配置作为默认启动项。但它不会立刻应用新配置。若要应用新配置，你需要重启系统，并进入最新生成的启动项。

`nixos-rebuild test` 会重构你的系统，同时立刻应用新配置，但它不会将新配置加入到启动菜单。

`nixos-rebuild build` 会从配置文件构建系统，同时在当前目录生成一个名为 `result` 的符号链接，指向 Nix Store 中的*衍生物*（derivation）。

`nixos-rebuild dry-activate` 会从配置文件构建系统，但不会应用之。相反，它会展示在应用新配置时的显著变化。

`nixos-rebuild build-vm` 会从配置文件构建 NixOS,并在 QEMU 虚拟机里运行。它会在当前目录留下一个名为 `result` 的符号链接，包含所构建的虚拟机。使用 `result/bin/run-<主机名>-vm` 以运行该虚拟机。


> 提示：你也可以使用 `--target-host` 参数来通过 SSH 远程重构其他设备上的系统，例如
>
> `nixos-rebuild --target-host user@example.org switch` 相当于在 `example.org` 上以 `user` 用户登录并执行了 `nixos-rebuild switch` 。
>
> 此命令会在你的设备上执行 NixOS 重构过程，并将其应用于目标远程设备。由于 NixOS 重构过程需消耗一定硬件资源，当目标设备硬件配置较低时，此方法尤为适用。
