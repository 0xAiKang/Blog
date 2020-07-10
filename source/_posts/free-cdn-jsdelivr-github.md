---
title: 免费 CDN：JsDelivr + Github
date: 2020-07-10 13:32:43
tags: Tutorial
categories: ["Tutorial", "GitHub"]
---

不知道大家通常是如何访问图床的，我之前一直使用的方式是：`GitHub` 图床 + `raw.githubusercontent`。

图片相关的资源全部放在`GitHub`上，然后使用GitHub 提供的素材服务器`raw.githubusercontent`去访问。但是这种方式存在一个问题，那就是放在 Github 的资源在国内加载速度比较慢，如果网络稍微差一些，资源可能就会加载失败。

因此需要使用 CDN 来加速来优化资源加载速度。

<!-- more -->

## CDN 是什么
> CDN的全称是`Content Delivery Network`，即内容分发网络。CDN是构建在网络之上的内容分发网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN的关键技术主要有内容存储和分发技术。——百度百科

由于某些原因，很多公用免费的 CDN 资源在中国大陆并不很好用，就算是付费的，也有一定的限制，例如每天的刷新次数有限之类的。
幸运的是在中国大陆唯一有 license 的公有 CDN竟然是免费的，它就是——[JsDelivr](https://www.jsdelivr.com/)。

## JsDelivr 是什么
> A free CDN for Open Source fast, reliable, and automated. —— JsDelivr 官网

根据官网的介绍我们可以知道它是一个**免费**、**快速**、**可靠**、**自动化** 的CDN。

那么，这么棒的CDN，到底该如何使用呢？下面会一一介绍。

## 快速上手
JsDelivr 目前有三种用法：
* Npm
* Github
* Wordpress

因为本文的重点是如何使用 GitHub + JsDelivr，来搭建免费的CDN，所以这里就不对其他两种用法做过多介绍。

### 1. 新建Github 仓库
这个仓库是用于存储资源文件的，最好是public，因为private的仓库，资源链接会带token验证，而这个token会存在过期的问题。

### 2. 将本地资源推送至仓库
将资源文件加入本地仓库，然后推送至 CDN 的远程仓库。

### 3. 发布仓库
如果没有发布就直接使用，可能会导致文件加载异常。

自定义发布版本号：
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200710132805.png)

然后点击`Publish release`。

### 4. 通过jsDeliver引用资源
只需要通过符合 JSDelivr 规则的 URL 引用，即可直接使用 Github 中的资源。

规则如下：
```
https://cdn.jsdelivr.net/gh/username/repository@version/file
```

参数说明：
* `cdn.jsdelivr.net/gh/`：jsDeliver 规定Github 的引用地址
* `username`：你的GitHub 用户名
* `repository`：CDN 仓库
* `@version`：发布的版本号
* `file`：资源文件在仓库中的路径

版本号不是必需的，是为了区分新旧资源，如果不使用版本号，将会直接引用最新资源，除此之外还可以使用某个范围内的版本，查看所有资源等，具体使用方法如下：
```
// 通过指定版本号引用
https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/avatar.jpg

// 使用一个范围内的版本
https://cdn.jsdelivr.net/gh/jquery/jquery@3.2.1/dist/jquery.min.js

// 忽略版本号则默认使用最新版
https://cdn.jsdelivr.net/gh/jquery/jquery/dist/jquery.min.js

// 在任意JS/CSS文件后添加 .min 能得到一个缩小版
// 如果它本身不存在，我们将会为你生成
https://cdn.jsdelivr.net/gh/jquery/jquery@3.2.1/src/core.min.js

// 在末尾加 / 则得到目录列表
https://cdn.jsdelivr.net/gh/jquery/jquery/
```

同样的一张图片，可以对比一下`jsDeliver`和`raw.githubusercontent` 的访问速度。
* `jsDeliver`：https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/avatar.jpg
* `raw.githubusercontent`：https://raw.githubusercontent.com/0xAiKang/CDN/master/blog/images/avatar.jpg



