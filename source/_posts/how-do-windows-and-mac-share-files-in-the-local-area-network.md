---
title: Windows 和 Mac 在局域网内如何共享文件？
date: 2020-08-11 22:11:18
tags: ["Skill", "Windows", "Mac"]
categories: ["Skill", "Windows", "Mac"]
---

每当手上有两台或多台电脑时，如果想传送一个文件，第一个想到的就是微信、QQ等这类工具。
如果碰到了大一点的文件，就得换成网盘或者移动硬盘。

身为一个做开发者，这种做法比较low，所以找了几篇文章学习到了如何在局域网内共享文件。

<!-- more -->

## 准备

这里准备的是用 Windows 作为主机创建共享文件。

首先要确认准备传输文件的 Windows 和 Mac 是在同一个路由器组成的局域网内。

然后打开 Windows 的文件资源管理器，在其根目录下创建一个共享文件夹，名称随意，自己知道就好了。

右键文件夹，点击属性，找到 共享 Tab，点击高级共享。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200811220638.png)

勾选共享此文件夹，点击确定。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200811220726.png)

然后回到共享文件夹，右键点击属性，找到共享，选择用户。

如果允许其他人写入，则选择 Everyone，更改为：读取/写入。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200811220755.png)

## 访问

Windows 本机访问
```
# ComputerName 表示：你的计算机名称
# ShareFolders 表示：共享文件夹名称
file://ComputerName/ShareFolders/
```

Mac 局域网访问

```
# ComputerName 表示：需要访问的计算机名称
# ShareFolders 表示：共享文件夹名称
smb://ConputerName/ShareFolders/
```
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200811220823.png)

通过验证之后，就能访问到共享文件夹了。

到这里应该就能顺利的在两个或多个电脑之间传输文件了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200811220855.png)

如果还不能访问，可以ping 一下对方的主机，如果没有ping通，检查一下防火墙设置。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200811220920.png)

### 参考链接
* [Windows 和 Mac 在局域网内如何共享文件？](https://zhuanlan.zhihu.com/p/32026197)
* [共享文件夹 一个实现Windows和Mac之间文件互传的简单方法](https://blog.csdn.net/sscssz/article/details/50057759)
