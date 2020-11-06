---
title: Mac 临时文件占用过多磁盘空间
date: 2020-11-05 20:09:58
tags: ["Mac", "Skill"]
categories: ["Linux"]
---

最近使用Mac 时，被告知磁盘空间严重不足了，我心想最近又没有下载什么大文件，怎么会突然满盘了。

<!-- more -->

于是使用DaisyDisk 扫描了一下磁盘空间，发现其中多达 186 G 全是临时文件。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/%E7%A3%81%E7%9B%98.jpg)

起初以为是系统产生的临时文件。因为并不知道这些文件是如何产生的，所以也不太敢直接删除，只尝试过重启电脑但并没有用。

后来通过Apple 社区提问才了解到，原来`achegrind.out` 这类文件全是 Xdebug 的输出文件！所以是可以直接删除掉的～

此前从未清理过这类文件，所以才会导致临时文件如此之大...

可以打开终端，使用如下命令进行清理：

```
sudo rm -rf /private/var/tmp/cachegrind.out.*

# 或者
sudo find /private/var/tmp -name "cachegrind*" -exec rm -rf {} \;
```

因为本地应用的Xdebug 一直都是开启着的，所以请求该应用时，Xdebug 就会将调试信息输出至临时文件了，如图：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201105140351.png)
