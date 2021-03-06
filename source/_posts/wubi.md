---
title: 98 版五笔字型基础教程 
categories: 
  - 五笔
  - 教程
author:
  - 一穗灯花
  - NixOS Community
date: 2022-07-19 19:06:00
tags:
  - 五笔
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

五笔字型（简称：五笔）是一种快速而准确的的中文形码输入法，但是需要记住很多东西。

五笔字型由王永民发明。


# 五笔字型的优势

如你所见，有很多方法可以在 NixOS 中输入中文，其中最常见的是拼音输入法。对于中文学习者而言，它不需要特殊学习，仅需要了解拼音（Pinyin, 中文汉字的罗马化转写）即可。这种基于汉字发音输入文字的方法叫作「音码输入法」。

与此同时，音码输入法存在一些问题：在中文里，同一个拼音可能会对应不同的汉字。这时，用户需要在输入法中选择正确的汉字。同时，拼音输入法往往会将那些使用频率较低的稀有汉字后置，这会让用户寻找它们的过程变得困难。

五笔字型是一种基于构成汉字的图形元素所设计的输入法，称为「形码输入法」。在五笔中，任何中文汉字的输入最多只需要四次点击键盘（在拼音输入法中，平均六次）。与此同时，一个汉字所对应的五笔输入码往往是唯一的，用户不需要从数个具有相同拼音的汉字中寻找正确的汉字。

通过本教程，您可以在约 30 分钟内精通 98 版五笔字型输入法。


# 安装 98 版五笔

在 NixOS 系统上，有两种方法可获得 98 版五笔输入法。
## 选项一：IBus - RIME
在 NixOS 配置文件中启用 IBus 输入法框架，以及适用于 IBus 的 RIME 输入法引擎
```nix
i18n.inputMethod = {
  enabled = "ibus";
  ibus.engines = with pkgs.ibus-engines; [ rime ];
};
```
然后从 Git 仓库中下载用于 RIME 的 98五笔输入法方案文件
```bash
 $ git clone https://github.com/yanhuacuo/98wubi.git
 $ mkdir -p ~/.config/ibus/rime/
 $ cp -r 98wubi/* ~/.config/ibus/rime/
```

## 选项二：使用 Fcitx5 - RIME
在 NixOS 的配置文件中启用 Fcitx5 输入法框架，以及适用于 Fcitx5 的 RIME 输入法引擎
```nix
  i18n.inputMethod = {
    enabled = "fcitx5";
    fcitx5.addons = with pkgs; [ fcitx5-rime ];
  };
```
然后从 Git 仓库中下载用于 RIME 的 98五笔输入法方案文件
```bash
 $ git clone https://github.com/yanhuacuo/98wubi.git
 $ mkdir -p ~/.local/share/fcitx5/rime/
 $ cp -r 98wubi/* ~/.local/share/fcitx5/rime/
```

完成之后，默认可以按 F4 或 Ctrl+` 调整基础设置。


> 提示：有关 RIME 引擎的进阶使用技巧，参见 [Rime (简体中文) - ArchWiki](https://wiki.archlinux.org/title/Rime_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

# 学习 98 版五笔

## 字根及其分类
字根，又称码元，是构成汉字的图形部件。

在学习五笔之前，我们需要认识字根图。

![](/images/zgen.jpg)


汉字由不同的字根组成，这些字根又是由笔画组成。所有字根均根据第一个笔画分为五个类别。每个类别映射到键盘上的一个区域，并且每个区域又有五个按键。因此，键盘包含 5 个区域和 25 个按键。

`````
 蓝色的区域（QWERT）称为「撇区」
 紫色的区域（YUIOP）称为「捺区」
 黄色的区域（ASDFG）称为「横区」
 绿色的区域（HJKLM）称为「竖区」
 橙色的区域（XCVBN）称为「折区」
`````
> 提示：在本文提供的输入方案中，使用 Z 键作为拼音反查键。

## 汉字的分类

在使用五笔前，需要了解汉字的分类。我们将汉字分为如下的类型：

```
0）键名字：
如上一部分中的图，每个键上着重标注的字根（如，Q 是「金」、G 是「王」），这种字称为键名字。我们将会在后文中详细说明这种字的输入方式。

1）上下型：
这种汉字由上下两部分组成。例如「苗」由「艹」和「田」组成；「型」由「刑」和「土」组成。这两个部分分别位于汉字的上方和下方。

2）左右型：
这种汉字由左右两部分组成。例如「汉」由「氵」和「又」组成；「明」由「日」和「月」组成。这两个部分分别位于汉字的左侧和右侧。

3）杂合型：
这种汉字分为 3 类：
  1. 连接类
  该组包括字根相互连接的汉字。例如「且」由「月」和「一」组成；「正」由「一」和「止」组成。这两个部分互相连接。
  2. 交叉类：
  该组包括字根相互交叉或重叠的汉字。例如「本」由「木」和「一」交叉；「申」由「日」和「丨」交叉。
  3. 半包围类：
  该组包括具有半包围结构的汉字。例如「赵」由「走」和「乂」组成；「辽」由「了」和「辶」组成；「间」由「门」和「日」组成。


4）成字码元：
这种汉字本身可作为一个字根，但又可以成为一个独立的汉字。例如「虫」，它是位于 J 键上的字根。可用于组成「蚊」等字的左半部分，也可用于单独构成「虫」字。我们将会在后文中详细说明这种字的输入方式。
```
> 提示：在阅读本文时，可以适当做些笔记。

## 拆字规则
拆字规则，是将汉字拆分为字根时所适用的规则。

1）尽量取更大的元素作为字根

![](/images/wubi-1.gif)

第一种拆字方法是正确的，不应将较大的字根「古」拆分为较小的「十」和「口」。

2）不要破坏笔画

![](/images/wubi-2.gif)

第一种拆字方法不正确，因为它破坏了竖线。第二种拆字方法也不正确，因为「旦」并非一个字根。

3）尽量避免笔画交叉

![](/images/wubi-3.gif)

第二种拆字方法不正确。汉字「天」是连接类的汉字，而通过这种拆分方式，我们错误地将它归类为「交叉类」。

> 提示：在打字时，键入字根的顺序是同样重要的，它与书写汉字的笔顺一致。主要为：先左后右；先上后下；先水平，后垂直；以此类推。

## 输入码规则

### 0）键名字：
我们先从最简单的部分开始。要输入每个键上的键名字，只需要连按相应的按键四次。

例如，连按 4 次 Q 可输入「金」；连按四次 G 可输入「王」。

### 1）字根数量大于或等于 4 的字：

四个字根的字可通过笔顺规则输入

![](/images/wubi-8.gif)

如果要输入包含大于 4 个字根数量的汉字，需要输入该汉字的前三个字根和最后一个字根。

例如「缩」可被拆分为「纟」、「宀」、「亻」和「日」。其中，「纟」、「宀」、「亻」分别是「缩」的前三个字根，「日」是最后一个字根。

### 2）成字码元
许多字根本身就是独立的汉字。要输入这样的汉字，首先按下它所在的键，然后输入它的前两个笔画和最后一个笔画。

![](/images/wubi-19.gif)

例如「手」，这个字根在 R 上，它的第一个笔画「丿」在 T 上，第二个「一」在 G 上，最后一个「丨」在 H 上。

### 3）字根数量小于 4 的字：

> 提示：这是对于新手而言最难的部分，

像「洒」、「沐」和「汀」这样的字，对应的字根所在的键是一样的，这就意味着输入它们的组合键相同。为了区分它们，五笔有了一种名为「识别码」的规则。在输入完这种汉字的所有字根之后，你还需要输入一个识别码。

一个汉字的识别码由两个因素决定：

1. 汉字最后一个笔画所的类别，包括横，竖，撇，捺，折。
2. 汉字的结构类型，包括上下型，左右型和杂合型。

例如，如果一个汉字的最后一笔是横，则该汉字的识别码位于横区。如果这个汉字同时是左右型，则识别码为 G；若它是上下型，识别码为 F；若为杂合型，则识别码为 D。

![](/images/wubi-20.gif)
![](/images/wubi-21.gif)

可通过下图辅助记忆

![](/images/sbm.png)

### 4）一级简码
只需按一下相应的键即可输入的 25 个常用汉字，被称为一级简码。可通过下图辅助记忆：

![](/images/wubi-10.gif)

> 提示：也可使用口诀记忆
>
> 我是中国人 QJKLW
>
> 在这上工的 DPHAR
>
> 和民同为主 TNMOY
>
> 产经要有地 UXSEF
>
> 不以一发了 ICGVB

# 使用 98 版五笔
> 子曰：「温故而知新，可以为师矣」。最好的记忆方法是复习。任何人在刚刚学习五笔时，都无法达到较快的速度。短时间内，也无法使用五笔在中方社交媒体上与人交谈。我们建议用户使用 98 版五笔字型练习输入中文文章。
