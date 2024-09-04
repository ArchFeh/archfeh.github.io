---
author: ArchFeh
title: 从零开始修复图形学bug
date: 2023-11-09
description: 
math: false
---

我是图形学<s>练习两个半月的实习生</s>，现在遇到了一个图形界面的bug，[terminal文字选中存在偏移](https://github.com/revyos/revyos/issues/11)。虽然学习了**OpenGL**，**Linux图形栈**等前置知识，但这是第一次直面bug，下面我将从几个方面来记录从零开始修复它的历程。

---

## 1. 确认问题，复现问题

当我们遇到一个用户提交的bug时，首先我们要确定这个问题能否复现，是不是真实存在且普遍存在。针对这个bug，我使用[Lichee Pi 4A](https://sipeed.com/licheepi4a)刷写了相同的固件，打开terminal，office，浏览器等软件，然后进行了选取操作，发现确实存在一些不跟手，没有指哪打哪的精准感。问题**确实**存在。

#### 对照参考

为了确定问题是出现在哪一部分。我选取了[VisionFive 2](https://www.starfivetech.com/en/site/boards)开发板作为对照组，查看在[VisionFive 2](https://www.starfivetech.com/en/site/boards)下是否存在同样的偏移，答案是否定的。我又试图使用[Lichee Pi 4A](https://sipeed.com/licheepi4a)刷写其他的系统固件，发现该问题还是存在。由此可以确定这是[Lichee Pi 4A](https://sipeed.com/licheepi4a)特有的问题。

---

## 2. 寻找解决方案

对于一个初学者来说，没法立刻定位到问题所在。这可能是Linux内核的问题，也可能是硬件的问题，也可能是图形驱动的问题或者桌面本身的bug。如果一个个测试的话，很费时费力，当然等以后经验老道了就可以一眼直击问题所在。

#### 沟通能力

我作为一个新人，我选择的方法不是闭门造车，而是和老手进行交流，来一场头脑风暴。在我们的工作中，个人能力固然重要，但是团队的力量也不可忽视。沟通能力至关重要，尤其是从事开源工作时，我们需要和不同语言不同文化的                     开发者沟通，在沟通时，我们要能够准确的描述问题。每个人的时间都很宝贵，如果浪费在理解你的话语上，就得不偿失了。我分别询问了两位同事，他们从两个角度给出了不同的可能性

- rv浮点操作问题
- 硬件指针问题

#### 独立思考

根据我们上文写的对照参考，我们可以初步得到一些判断，小概率是rv浮点操作问题，大概率是硬件指针问题。我遵循由表及里的步骤来解决问题，越是表层的问题，排除起来越容易，越节省时间。

## 3. 开始解决问题

为了解决硬件指针的问题。我们需要查阅Linux图形栈里硬件指针属于哪一部分。既然bug是鼠标指针偏移，那就可以得到 `X.Org——>InputDevice` 的链条，我们来查阅有关 `InputDevice` 的男人（man）文件，我们直接 `$ man xorg.conf` ,可以看到

> **InputDevice** sections have the following format:
> 
> <pre><b>Section &#34;InputDevice&#34;</b>
> <b>    Identifier &#34;</b><i>name</i><b>&#34;</b>
> <b>    Driver     &#34;</b><i>inputdriver</i><b>&#34;</b>
> <i>    options</i>
> <i>    ...</i>
> <b>EndSection</b></pre>

在之前对xorg的学习中，我们知道常用的input driver有[libinput](https://wiki.archlinux.org/title/libinput),但是在libinput的配置文件中，我们没有找到有关指针的选项。但是除libinput之外，还有一个 `modesetting` 驱动，在 `$ man modesetting` 之后，我们可以看到如下的配置

> The following driver **Options** are supported:
> [ **Option "SWcursor" "**  *boolean* **"**](https://man.archlinux.org/man/modesetting.4#Option)
> Selects software cursor. The default is **off.**

这不就是我们的软件指针设置吗？立刻

```
$ vim /usr/share/X11/xorg.conf.d/10-modsetting.conf
Section "Device"
	Identifier "PVR"
	Driver "modesetting"
	Option "SWCursor" "true"
EndSection
```

重启xorg，发现问题解决了！此问题的解决方法就是将硬件指针关闭，切换为软件指针。但是这不是一种普适的解决方法，一般我们很少会使用软件指针，而且硬件指针的使用体验会比软件指针更好。modesetting驱动又与内核的ioctl有关，又可以合理的怀疑是Linux内核方面存在问题，当然这些是后话。

## 总结

所以当我们进入一个新的领域，面对新问题的时候，我们解决问题只需要遵循如下的方法论：

- 确认问题
- 多沟通，破除信息差
- 独立思考，合理推测
- 由表及里，逐个击破

