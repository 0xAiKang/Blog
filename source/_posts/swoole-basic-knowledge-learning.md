---
title: Swoole 基础知识学习
date: 2020-11-03 22:33:46
tags: ["PHP", "Swoole"]
categories: ["PHP"]
---
这篇笔记用来记录Swoole 基础知识的学习。

<!-- more -->

## Master、Manager、Worker、Reactor
### Master
Master 进程是一个多线程进程。

### Manager 进程
负责创建 / 回收 worker/task 进程

### Worker 进程
* 接受由 Reactor 线程投递的请求数据包，并执行 PHP 回调函数处理数据
* 生成响应数据并发给 Reactor 线程，由 Reactor 线程发送给 TCP 客户端
* 可以是异步非阻塞模式，也可以是同步阻塞模式
* Worker 以多进程的方式运行

### Reactor 线程
* Reactor 线程是在 Master 进程中创建的线程
* 负责维护客户端 TCP 连接、处理网络 IO、处理协议、收发数据
* 不执行任何 PHP 代码
* 将 TCP 客户端发来的数据缓冲、拼接、拆分成完整的一个请求数据包

有一个更加通俗的比喻来描述这三者的关系：
假设 `Server` 就是一个工厂，那 `Reactor` 就是销售，接受客户订单。而 `Worker`就是工人，当销售接到订单后，`Worker`去工作生产出客户要的东西，而 `TaskWorker` 可以理解为行政人员，可以帮助 `Worker` 干些杂事，让 `Worker`专心工作。

## 其他

IPv4 使用 127.0.0.1 表示监听本机，0.0.0.0 表示监听所有地址
IPv6 使用::1 表示监听本机，:: (相当于 0:0:0:0) 表示监听所有地址

### TCP 协议
TCP (Transmission Control Protocol 传输控制协议）协议是一种面向连接的，可靠的，基于字节流的传输通信协议。

### UDP 协议
UDP (User Datagram Protocol 用户数据报协议）是一种无连接的传输层协议，提供面向事务的简单不可靠信息传送服务。

UDP 服务器与 TCP 服务器不同，UDP 没有连接的概念。启动 Server 后，客户端无需 Connect，直接可以向 Server 监听的 9502 端口发送数据包。

## 常见问题

### TCP “粘包”问题
首先来解释以下所谓的“粘包”问题其本质是什么。

服务端建立服务，客户端向服务端发起连接，正常情况下，服务端的每次 send，客户端都能正常 recv。但在并发的情况下，服务端的两次send 或者更多次 sned，客户端可能一次就 recv了。

所以这就导致“粘包”问题的产生。

TCP 协议的本质是流协议，它只会保证保证发送方以什么顺序发送字节，接收方就一定能按这个顺序接收到。所以所谓的“粘包”问题不应该是传输层的问题，而是应用层的问题。

### 无法连接到服务器的简单检测手段
1. 在 Linux 下，使用 `netstat -an | grep` 端口，查看端口是否已经被打开处于 Listening 状态
2. 上一步确认后，再检查防火墙问题，这里的防火墙指的是机器本身的防火墙，如果是云服务器，那么还包括云的防火墙。
3. 注意服务器所使用的 IP 地址，如果是 `127.0.0.1` 回环地址，则客户端只能使用 `127.0.0.1` 才能连接上，所以如果希望其他机器也能访问本机，那就使用`0.0.0.0`。

### 参考链接
* [怎么解决TCP网络传输「粘包」问题？](https://www.zhihu.com/question/20210025)