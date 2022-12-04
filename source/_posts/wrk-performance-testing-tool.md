---
title: Wrk 性能测试工具
date: 2022-12-04 10:57:25
tags: ["Linux", "Mac"]
categories: ["Linux"]
---

常用的性能测试工具，Apache ab 这个用得比较多的，这篇笔记用来介绍另外一个轻量级性能测试工具——[wrk](https://github.com/wg/wrk)。

<!-- more -->

> wrk 是一款针对 Http 协议的基准测试工具，它能够在单机多核 CPU 的条件下，使用系统自带的高性能 I/O 机制，如 epoll，kqueue 等，通过多线程和事件模式，对目标机器产生大量的负载——Github
> 

wrk 的优势：
1. 轻量级的性能测试工具，开箱即用
2. 安装简单，各大操作系统基本上都能一键安装
3. 学习成本很低，记住常用的几个参数即可
4. 基于系统自带的高性能 I/O 机制，如 epoll, kqueue, 利用异步的事件驱动框架，通过很少的线程就可以压出很大的并发量

## 安装

### MacOS
wrk 的安装非常简单：
```bash
$ brew install wrk
```

### Ubuntu

```bash
$ sudo apt-get update
$ sudo apt-get install wrk
```

验证是否安装成功：
```bash
$ wrk -v
wrk 4.2.0 [kqueue] Copyright (C) 2012 Will Glozer
```

## 使用方法

wrk 命令格式如下：
```bash
$ wrk <options> <url>
```

其中常用参数如下：
* -c, --conections: 跟服务器建立并保持的 TCP 连接数量
* -d, --duration: 压测时间，默认为 10s
* -t, --threads: 使用多少个线程进行压测（线程数一般是核数的 2 到 4 倍，过多会出现线程切换过多导致效率降低）
* -H: 指定请求的 HTTP Header，有些 API 需要传入一些 Header，可通过 Wrk 的 -H 参数来传入
* --latency: 在压测结束之后，打印延迟统计信息
* -T --timeout: 请求超时时间
* -v，--version: 打印正在使用 wrk 详细版本信息
* -s, --script: 指定 Lua 脚本路径

## 测试报告

利用 wrk 对 `www.baidu.com` 发起压力测试，线程数为 4，模拟 300 个并发请求，持续 30 秒，并在压测结束之后，打印延迟统计信息。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221202090629.png)