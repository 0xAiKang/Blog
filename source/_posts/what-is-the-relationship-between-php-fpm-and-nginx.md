---
title: PHP-FPM 与 Nginx 是什么关系？
date: 2020-09-23 22:35:17
tags: ["PHP", "PHP-FPM", "Nginx"]
categories: ["PHP"]
---

最近部署了几次项目，经常遇到这样一个错误：`Nginx 502 bad gateway`，查看 Nginx 错误日志之后，发现这样一段话：`Primary script unknown`，找了好久的答案，总结出以下几个原因：

<!-- more -->

* 未启动 Nginx 
* 未启动 php-fpm
* Nginx 配置异常
* 文件夹权限不足

其中未启动 php-fpm 是出现最多的错误，再聊 php-fpm 之前，我们先来学习几个 相关概念。

### 什么是 cgi
Cgi 是一个协议，它约定了 web server 和应用程序（如：PHP、Python等）之间的信息交换的标准格式。

#### 静态文件

当一个客户端试图访问`index.html`这个文件时，那么 web server 就回去文件系统中找到这个文件，最后将结果返回给客户端。

#### 非静态文件

当一个客户端试图访问`index.php`这个文件时，web server 收到请求之后，根据配置文件知道了自己处理不了，接着转发给第三方的应用程序（PHP解析器、Python解析器等），web server 知道该传哪些数据吗？它不知道，**所以 Cgi 就是约定要传哪些数据，以什么样的格式传递给第三方的应用程序的协议。** 应用程序独立处理完该脚本，然后再将结果返回给产生响应的 web server，最后转发响应至客户端。

当 web server 收到 `index.php` 这个请求之后，会启动对应的 cgi 程序（PHP解析器，Python解析器），接下来解析器会解析 php.ini 配置文件，初始化执行环境，然后处理请求，再以 cgi 规定的格式返回处理后的结果，退出进程。web server 将转发响应至客户端。

这种协议看上去简单有效，但它也存在一些明显不足：
1. 每一个请求产生唯一一个进程，从一个请求到另一个请求，内容和其他的信息全部丢失。
2. 开启一个进程会消耗系统的资源，大而重的并发请求（每产生一个进程）数量很快会使服务器一团糟。

### 什么是 fastcgi
知道了 cgi 是协议之后，那 fastcgi 又是什么呢？

知道了 cgi 服务器性能低下的原因是因为每产生一个请求，都会做同样的事情：解析器解析配置文件，初始化执行环境，启动一个新的进程。

fastcgi 则是在 cgi 的基础上做了重大的改进，从而达到相同的目的，原理如下：
1. fgstcgi 使用了能够处理多个请求的持续进程，而不是针对每个请求都产生新的进程。
2. fastcgi 是一个基于套接字的协议，因此它能够适用于任务平台（web server）及任何编程语言。

fastcgi 的性能之所以高于 cgi，是因为 fastcgi 可以对进程进行管理，而这是 cgi 所做不到的，但它的本质仍然是 协议。

### 什么是 php-fpm
默认情况下，PHP 是支持 cgi 和 fastcgi 协议的。

PHP 二进制命令能够处理脚本并且能够通过套接字与Nginx 交互，但是这种方式并不是效率最高的，php-fpm 便是在这样的背景下诞生的。

PHP-FPM （PHP FastCgi 进程管理，PHP Fastcgi Process Manager）

php-fpm 将 fastcgi 带到了一个全新的水平。

### php-fpm 和 nginx 有什么联系
在理解了 cgi、fastcgi、php-fpm 是什么之后，就不难理解 php-fpm 和nginx是什么关系了。

因为 php-fpm 是 php fastcgi 的进程管理器，所以 php-fpm 就是 nginx 与 php 交互时，协助 php 将性能发挥最大的一个程序。

难怪每次 php-fpm 这个进程死掉时，nginx 的状态就变成了 502 。

### 参考链接
* [搞不清 Fastcgi 和 cgi 关系](https://segmentfault.com/q/1010000000256516)
