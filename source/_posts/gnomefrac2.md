---
title: 后传：让 NixOS 的 GNOME X11 会话实现 150% 缩放
categories: 
  - NixOS
  - 教程
#pic: https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/NixOS_logo.svg/1280px-NixOS_logo.svg.png
author: 
 - 一穂灯花
date: 2023-11-23 11:40:00
tags: 
 - NixOS
 - Linux
---

<h1 align="center">
  <img src="https://pic.lanta.cyou/img/nix-snowflake.svg" width="200">
  <br>NixOS<br>
</h1>

<div class="info">

# 前言

在我之前的文章《杂谈：让 NixOS 的 GNOME X11 会话实现 150% 缩放》中，提到了一个在 NixOS 上使用 overlay 为 GNOME 打补丁以实现非整数倍缩放的方式。然而考虑到不同读者使用的 GNOME 版本不同。故在这里写出通用性的办法。

# 正文

1. 打开 https://github.com/puxplaying/mutter-x11-scaling/tree/45a4855cec7064a494712ed2c9996cf12d0633aa ，然后浏览它的 commit 历史。查看其中适用于你所使用的 GNOME 版本的 commit。

2. 选择你所使用的 GNOME 版本，打开 PKGBUILD 文件，记录 scaling_commit

3. 将 scaling_commit 替换入 NixOS overlay 中的 commit。

# 示例

以下是下一个 GNOME 44.5 的例子。

打开 https://github.com/puxplaying/mutter-x11-scaling/blob/45a4855cec7064a494712ed2c9996cf12d0633aa/PKGBUILD ，在第 63 行看到了如下内容：

```
_scaling_commit=c71847d5e7f2e08e8bf4e81257c24bbcd422d355
```
然后打开 NixOS overlay 文件，这里文件名为 mutterX11.nix


```
#mutterX11.nix

final: prev: {
  gnome = prev.gnome.overrideScope'
    (final': prev': {
      mutter = prev'.mutter.overrideAttrs (old: {
        patches = (old.patches or []) ++ [
          (final.fetchpatch {
          # 注意，下面的 c71847d5e7f2e08e8bf4e81257c24bbcd422d355 需要换成你所需要版本 GNOME 的。
	    url = "https://salsa.debian.org/gnome-team/mutter/-/raw/c71847d5e7f2e08e8bf4e81257c24bbcd422d355/debian/patches/ubuntu/x11-Add-support-for-fractional-scaling-using-Randr.patch";
	    
	    # 如果不知道 sha256 可以留空，然后 nixos-rebuild 的时候，系统会自动算出 sha256。
	    sha256 = "sha256-JUTWgQWQmevAfGvNn9XPMvFQkkAj9JK9rZ4piAxlJ8E=";
	  })
        ];
      });
    });
}
```

最后，在 configuration.nix 中应用 overlay。

```
#configuration.nix 

  nixpkgs.overlays = [
    (import ./mutterX11.nix)
  ];


```