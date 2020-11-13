---
title: Google Drive 如何转存文件？
date: 2020-11-13 21:55:50
tags: ["Google Drive", "Tutorial"]
categories: ["Skill"]
---

对于初次使用[Google Drive](https://www.google.com/drive/)（以下简称 GD）的同学来说，可能会有以下几点困惑。

<!-- more -->

1. 大家常说的转存是什么意思？
2. 常见的转存方式有哪几种？

在正式回答上面两个问题之前，先来了解一下GD。

GD 是Google 在2012 年4 月24 日推出的一个在线同步存储服务，类似百度的百度网盘，不过不同之处在于GD 不会限速。普通用户默认的存储空间是15 GB。

用户可以将其他用户分享的文件添加到“我的云端硬盘”，这种方式并不会占用用户的存储空间，这个操作相当于是在“我的云端硬盘”中创建了一个**软链接**，可以快速访问该文件，而文件所有者则还是**分享者**，如果原作者删除了，那么你网盘里的也会消失。

所以为了解决上述问题，转存的概念便诞生了，它存在的意义是将其他用户分享的文件保存至自己的云盘，类似百度网盘的“保存到我的网盘”功能。

但有所不同的是，如果**分享者**没有开放权限，那么其他用户则无权转存。

### 方式一
在需要转存的文件上，点击右键，制作一个拷贝，拷贝的文件位于“我的云端硬盘”中。

第一种方式最简单，适用于小文件，不能对文件夹进行 Copy 操作。

### 方式二
[Copy, URL to Google Drive](https://softgateon.herokuapp.com/urltodrive/) 是一个云端硬盘插件。

在目标文件上点击右键，选择打开方式，关联更多应用。

搜索Copy, URL to Google Drive 进行安装。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201113214943.png)

安装完成之后，还需要进行Google 账号授权才能进行转存操作。

在需要转存的文件夹上 右键-打开方式-Copy, URL to Google Drive，之后点击 Save, Copy to Google Drive，就可以看见正在转存了，如果文件较大时间会比较久。

### 方式三
在Telegram 上有人开发了一个机器人（@GoogleDriveManagerBot），专门用于GD 文件转存。

该机器人可以实现谷歌网盘资源转存以及网盘内资源批量重命名，普通用户仅可绑定一个 GD 账号。
通过简单的命令即可对文件进行转存。

### 总结
方式一最简单，门槛最低，即使在没有权限的情况下，也能进行Copy 操作，但是效率很低。

方式二、三省事，效率高，但前提是得有权限。

### 参考链接
* [一个方便转存 Google Drive 分享文件的方法](https://www.jianshu.com/p/42f323bd15ef)
* [转存Google Drive资源到自己的Google Drive](https://blog.curlc.com/archives/569.html)
* [Linux 下使用 rclone 挂载网盘到本地](https://blog.frytea.com/archives/31/)