---
title: 手把手教你如何启动AWS实例
date: 2020-08-10 14:05:17
tags: ["Tutorial","Linux"]
categories: ["Tutorial", "Linux"]
---

> 什么是AWS ？

[Amazon Web Services ](https://aws.amazon.com/cn/?nc2=h_lg)(AWS) 是亚马逊提供的全球最全面、应用最广泛的云平台。

<!-- more -->

云这个概念最开始是从国内的阿里云、腾讯云这些地方听到的，后来服务器接触的多了，也慢慢了解了一些国外的云，如：亚马逊云、微软云。

创建一台实例其实是非常简单的事情，但由于这方便资料比较少，导致对于新用户可能不那么友好，我自己当初创建时就不怎么顺利。所以整理这篇笔记的目的有两个，一是方便自己日后回顾，二是给第一次使用的用户一些参考。

## 启动实例
首先登入到AWS ，找到[EC2](https://console.aws.amazon.com/ec2/v2/home) 并点击

![AWS](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810115939.png)

在左侧菜单栏中点击实例

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810121213.png)

点击启动实例

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810121240.png)

## 配置实例
选择系统映像，这里以Linux 操作系统为例，我选择是`Ubuntu Server 18.04 LTS`，这个版本表示Ubuntu 服务端 长期稳定支持版本。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810121309.png)

选择实例类型，根据自身需要考虑，当然 性能越好价格越高。这里我选择的是一个中等偏下的类型。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810134659.png)

配置实例详情信息，这里的这些核心配置，通常都保持默认，只是将自动分配公有IP 地址改为禁用。这样再重启机器时，就不会改变IP了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810134802.png)

根据自身需要分配合适的硬盘大小。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810134833.png)

配置安全组，所谓安全组就是拥有相同防火墙规则的群组。这个也是根据自身需要选择是否共用同一个安全组。

拥有同一个安全组就表示拥有相同的防火墙规则。设置完安全组之后，点击审核和启动。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810134858.png)

下面会有一个界面给你确认机器的配置是否无误的，从头到尾检查没有问题之后就可以点击启动实例了。

## 创建密钥
可以选择共用已有的密钥对也可以选择新建一个。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810134919.png)

然后点击启动实例。

## 分配弹性IP
启动完成之后点击查看实例。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810134956.png)

在实例列表中，找到该实例之后，分别点击操作=>联网=>管理IP 地址=>分配弹性 IP

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810135020.png)

确认分配

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810135056.png)

分配成功之后，会得到一个弹性IP（公有），然后返回实例列表

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810135121.png)

## 关联IP 地址

找到刚才启动的那个实例（没有实例ID），分别点击操作=>关联地址

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810135143.png)

这一步很重要，这里要将实例和弹性IP 地址关联，所以要选择该弹性IP 对应自己的实例。如果不确定是哪一个，可以返回到实例列表中去查看，就是那个没有名称的实例。

然后点击关联

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810135205.png)

关联成功

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810135233.png)

直到做完这一步才算正真的启动好一个实例。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200810135302.png)

### 连接
启动好实例之后，如何连接呢？

```
$ ssh -i <私钥路径> ubuntu@ipaddress
```

指定刚才生成的密钥对，使用ssh命令 即可连接。