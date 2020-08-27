---
title: 手把手教你如何创建启动 Google Cloud 实例
date: 2020-08-26 23:25:21
tags: ["Linux", "云", "Tutorial"]
categories: ["Linux", "Tutorial"]
---

最近需要在Google Cloud 上重新开一台Hk区的服务器，所以写这篇笔记用来记录操作过程。

<!-- more -->

## 创建VM 实例
* [Google Cloud 官网](https://cloud.google.com/)
* [Google Cloud Platform 控制台](https://console.cloud.google.com/)

进入控制台，找到 Compute Engine，点击创建实例。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827142145.png)

新建虚拟机实例，选择相应的配置。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827142211.png)

选择操作系统映像，以及磁盘大小。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827142232.png)

基本配置如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827142253.png)

然后点击创建就可以了。创建成功之后，就可以看到该服务器的IP地址了。

这里需要注意的是，Google Cloud 的远程连接SSH 的方式与其他平台有所区别。

## 创建SSH 连接
Compute Engine =》元数据 =》SSH 密钥

找到修改，然后上传你的 SSH Key。

不知道SSH Key 是什么？

```
$ ssh-keygen -t rsa
# 打开终端，输入上面那个命令
# 然后在~/.ssh 目录下会生成一个 公钥和私钥
# 将 .pub 结尾的文件打开，复制其中的值，粘贴到Google Cloud 上就可以了。
```

### 连接

使用`ssh -i max@35.241.77.3` 命令连接，其中 max 是用户名，后面是对应服务器 ip地址。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827142317.png)

## 参考链接
* [开启Linux 虚拟机使用快速入门--官网文档](https://cloud.google.com/compute/docs/quickstart-linux)
* [GCP（Google Cloud Platform）入门](https://zhuanlan.zhihu.com/p/40983101)