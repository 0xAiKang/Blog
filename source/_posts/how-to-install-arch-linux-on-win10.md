---
title: how to install arch linux on win10
date: 2020-08-16 17:50:09
tags: ["Linux", "WSL", "Arch Linux", "Windows"]
categories: ["Linux", "Windows"]
---

最近主力生产工具可能要拿去送修，所以可能有一段时间要和我的MBP 分开了。但是工作还是要继续，于是把之前闲置的 小米 Pro 15.6 给整起来。

第一件需要做的事情就是配置开发环境。

<!-- more -->

## 了解 WSL
###  什么是 WSL ？

Windows Linux Server (WSL) 又名Windows 子系统，它使得开发人员可以直接在未经修改得Windows 上运行 `Gun/Linux` 环境，也包括大多数命令行工具，实用程序员和应用程序员，而不会需要额外增加虚拟机。

### WSL 可以做什么
* 你可以自行选择你喜欢的 `Gun/Linux` 发行版：Arch Linux、Ubuntu、OpenSuSE、Kail Linux、Debian、Fedora等。
* 运行通用的命令行，例如grep，sed，awk或其他ELF-64二进制文件。
* 轻松运行Bash Shell脚本和 `GNU/Linux` 命令行应用程序
* 使用自己的 `GNU/Linux` 分发程序包管理器安装其他软件。
* 使用类似Unix的命令行外壳调用Windows应用程序。
* 在Windows上调用 `GNU/Linux` 应用程序。

有了这些功能，我们就可以完成很多工作，而不必担心安装虚拟机监控程序，从而享受Linux的好处。安装并准备好Win 10后，请按照以下步骤进行操作，并在其中添加Arch Linux。

## 安装 WSL
本文要安装的WSL 是 [Arch Linux](https://www.archlinux.org/) 。

> 为什么要选择 Arch Linux？

因为它是一个轻量级且灵活的Linux 发行版。

### 为Linux 安装Windows 子系统
这是一项使Windows能够“ 托管 ” Linux 的功能。所以需要先启用此功能。

以管理员的身份打开Power Shell，然后输入以下命令：
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
通常会重启一次你的电脑。

### 安装Arch Linux 
我记得在2019 年，Windows 刚拥抱 Linux 时，Arch Linux 还可以直接从 Microsoft Store 直接下载，不知为何现在却搜不到了。

不过还是有其他办法手动安装，[打开该页面](https://github.com/yuk7/ArchWSL/releases/tag/20.4.3.0)，下载`Arch.zip`。

解压完成之后，可以看到如下文件：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200816171455.png)

双击`Arch.exe`应用程序，进行安装。

稍微等待一会，就可以看到Arch Linux 已经顺利安装完成了，然后按任意键退出。

### 启动Arch Linux
再次双击`Arch Linux`，不出意外的话，就可以看到`Arch Linux` 的控制台了，没错就是这么简单。

## 配置
第一次安装完成之后，需要手动做一些配置，初始化并更新系统。

在终端或`CMD` 中输入`WSL` 进入`Arch Linux`。

编辑 `/etc/pacman.d/mirrorlist`，去掉`China`节点 前面的`##`，以及下面的`Server`下面的`##`。

### 初始化
```
pacman-key --init

pacman-key --populate archlinux
```

### 更新
```
// 更新 GPG key
pacman -Sy archlinux-keyring

// 更新系统，速度快慢与镜像源有关
pacman -Syyu base base-devel 
```

## 个性化
Arch Linux 默认的样式并不好看，和CMD 都是黑漆漆的一片。

因为Arch Linux 默认使用的 Bash，如果你和我一样，更喜欢 Zsh 的话，那就请继续看下去。

### 安装ZSH
既然要安装Zsh，那就不得不安装`oh-my-zsh`了，所以这里一起安装了。
```
pacman -S zsh oh-my-zsh-git
```

### 安装Spaceship ZSH
[Spaceship ZSH](https://github.com/denysdovhan/spaceship-prompt) 是Zsh 的提示符工具。

1. 克隆仓库
```
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"
```

2. 链接文件
```
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```

3. 更改默认theme 
```
# vim ~/.zshrc
ZSH_THEME="spaceship"
```

重启终端即可。

### 参考链接
* [安装ArchWSL（Windows 下的Arch Linux 子系统）](https://github.com/way-zer/way-zer.github.com/issues/2)