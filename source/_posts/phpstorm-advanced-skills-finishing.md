---
title: PHPStrom 高级技巧整理
date: 2020-10-01 10:39:34
tags: ["PHP", "PHPStorm"]
categories: ["PHP"]
---

PHPStrom 是我日常使用频率很高的 IDE。

基础的使用这里就不过多介绍了，这里主要是用来整理一些比较高级的用法。

### 调试技巧

调试是日常开发中，不可缺少的一部分。

#### 路径映射
通常都是用来调试本地代码，可如果需要调试虚拟机或者其他应用中时，那该怎么做呢？

打开偏好设置或者设置，找到一个已经配置好的服务，勾选映射。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200930144728.png)

然后找到该服务的入口配置文件，后面的文件路径填绝对路径。