---
title: Win10 如何卸载 Arch Linux 
date: 2020-08-16 17:56:22
tags: ["Linux", "Arch Linux"]
categories: ["Linux", "Windows"]
---

最近在Windows 上安装 WSL，遇到一点问题，需要将 Arch Linux 完全卸载。

<!-- more -->

在正式卸载之前，有以下几点需要注意：
1. 不要试图通过 Microsoft Store 去卸载，那里只有安装按钮，没有卸载按钮。
2. 秋季创意者更新之前，可以使用`lxrun`命令去进行卸载操作，但是秋季创意者更新之后该命令就被移除了。

### 查看发行版

列出当前已经安装且随时可用的发行版：
```
wslconfig /list
```

列出所有发行版，包括正在安装、卸载和已损坏的发行版：
```
wslconfig /list /all
```

### 卸载

卸载已经安装的发行版：
```
$ wslconfig /list /all
Windows Subsystem for Linux Distributions:
Arch (Default)
$ wslconfig /unregister Arch
Unregistering...
```
上面是以`Arch Linux`为例进行卸载，其他发行版同理，只需要替换发行版的名称就可以了。

> 注意: 卸载发行版时，会永久删除所有与该发行版有关的数据和设置。

### 参考链接
* [Windows 10 Linux子系统如何卸载？](https://blog.littlelanmoe.com/exp/494)