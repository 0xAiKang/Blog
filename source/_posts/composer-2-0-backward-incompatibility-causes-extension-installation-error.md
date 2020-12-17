---
title: Composer 2.0 向下不兼容导致扩展安装出错
date: 2020-12-17 20:40:38
tags: ["PHP", "Composer"]
categories: ["Linux"]
---

今天在部署服务器环境时，遇到一个由Composer 版本向下不兼容而引发的问题，记录一下。

<!-- more -->

## 问题描述
后台Api 应用是用`ThinkPHP6.0` 的多应用模式开发的，起初部署时，总是提示找不到控制器。

当时就比较郁闷，怎么会找不到控制器呢？这个异常通常只会在没有开启多应用模式时才会出现，可是我明明已经开启了多应用模式，也安装了相关扩展（Composer 2.0.x 执行 composer install 没有直接抛出异常）。

正当我百思不得其解时，不经意间看到了我目前所使用的 Composer 版本是 `2.0.x`。

回头对比了一下我本地的版本：`1.8`，Google 一下才发现Composer 2.0 系列是最近才发布的，于是马上就想到了是否是 Composer 向下不兼容导致。

好家伙，真的是兼容性导致的问题：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201217105443.png)

## 解决办法
既然是版本过高导致的兼容性问题，那就好办了，直接降低版本即可。

Composer 降级非常简单，不用重新编译安装，直接使用以下命令即可：
```
composer self-update 1.8.0
```

如果你不知道有哪些版本可选择，可以查看官方的[发布历史](https://getcomposer.org/download/)。

### 参考链接
* [ThinkPHP V6.0.5版本发布——兼容Composer2.0](https://blog.thinkphp.cn/1997806)
* [Composer 中文文档](https://www.kancloud.cn/thinkphp/composer)