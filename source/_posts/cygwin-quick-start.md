---
title: cygwin quick start
date: 2020-09-01 22:16:25
tags: ["Cygwin", "终端"]
categories: ["终端"]
---

在很早之前就听说过`Cygwin`和`MinGW64`这两东西。

只是当时不是很理解这两个东西是做什么的，还经常和`msysGit` 搞混淆。

加上最近用`MinGW64`用的很不顺手，所以打算安装一个`Cygwin`。

## 区别与联系

首先来介绍下这三者分别是什么。

### Cygwin
Cygwin是一个类似Unix的环境和Microsoft Windows命令行界面。

大量GNU和开源工具，提供类似于 Windows上的 Linux发行版的功能。用官网的话说就是：在Windows 上获取Linux 的感觉。

### MinGW64
MSYS(MSYS | MinGW) 是一个在 Windows 下的类`Unit`工作环境。因为 Git 里面包含很多 Shell 跟 Perl 脚本，所以它(Git)需要一个这样的环境。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200901221056.png)

每次右键打开`Git Bash`时，其终端就是`MinGW64`

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200901221113.png)

### msysGit
msysGit是一个构建环境，其中包含希望通过为Git for Windows编写代码来贡献所需的所有工具。

所以，Git for Windows 可以在 Windows 上安装可运行 Git 的最小环境，而 msysGit 是构建 Git for Windows 所需的环境。

## 安装Cygwin
安装Cygwin 的过程比MinGW 要复杂些，其中主要需要注意的是模块部分。

Cygwin 好用的原因很大程度上是因为其功能之丰富，而各种功能则是来自于其模块。

终于安装好了，感觉很厉害的样子，是我想要的东西，希望在今后的日子中 能和它好好相处。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200901221141.png)

Mintty是一个终端仿真器 用于Cygwin的， MSYS或 Msys2 和衍生的项目，以及用于WSL。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200901221152.png)

### 参考链接
* [从Windows 运行下载Cygwin64](http://mingw-w64.org/doku.php/download/windows)
* [Cygwin 是什么，不是什么？--官网](https://cygwin.com/index.html)
* [Cygwin 安装教程 详细](https://www.crifan.com/files/doc/docbook/cygwin_intro/release/html/cygwin_intro.html#install_cygwin_setup_exe)