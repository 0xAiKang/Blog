---
title: Windows 如何安装 Swoole？
date: 2020-10-21 19:56:53
tags: ["PHP", "Windows", "Cygwin", "Swoole"]
categories: ["PHP"]
---

[Swoole](https://www.swoole.com/) 是一个 PHP 的协程高性能网络通信引擎。

<!-- more -->

目前仅支持 Linux(2.3.32 以上内核)、FreeBSD、MacOS 三种操作系统，它并不支持直接在 Windows 下安装，因为Windows 系统默认没有以下软件：
* gcc-4.8 或更高版本
* make
* autoconf

如果一定要在Windows 系统中使用，则可以使用 [CygWin](http://cygwin.com/) 或 WSL(Windows Subsystem for Linux) 。

这篇笔记并不介绍如何在Windows 系统中，安装Cygwin，如果需要，可以参考[Cygwin 快速上手](https://www.0x2beace.com/cygwin-quick-start/) 。

需要注意的是，在安装Cygwin 时，记得勾选以下软件包：
* gcc、gcc++
* autoconf
* php-devel
* pcre2 

### 安装Swoole
1. 可以通过以下方式下载 Swoole
* [github](https://github.com/swoole/swoole-src/releases)
* [pecl](https://pecl.php.net/package/swoole)
* [gitee](https://gitee.com/swoole/swoole/tags)

2. 从源码编译安装

下载源代码包后，将其拷贝至 Cygwin 的home 目录，解压并进入文件夹。
```
tar -zxvf swoole-src.tgz
```

编译安装：
```
cd swoole-src && \
phpize && \
./configure && \
make && sudo make install
```

如果因为某个软件包缺失而导致编译安装失败，则可以重新安装 Cygwin（重新安装不用卸载之间的版本，直接在此安装就好了）。

3. 启用扩展

编译安装到系统成功后，需要在 php.ini 中加入一行 `extension=swoole.so` 来启用 Swoole 扩展。

需要注意的是，通过这种方式安装的Swoole，最终存在于Cygwin 环境中，与宿主机中的PHP 版本无关。

通过`php -m | grep  swoole`查看是否安装成功。
