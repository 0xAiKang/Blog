---
title: PHP Socket 编程
date: 2020-10-15 22:29:16
tags: ["PHP", "Socket"]
categories: ["PHP"]
---

最近因为一些原因接触到一个老古董项目，这个项目虽然有些老，但仔细看一看，还是能学到一些东西的。

关于 PHP Socket 编程的文章有很多，这里就只简单记录一下如何快速上手。

<!-- more -->

## 什么是 Socket
按照惯例，还是先来了解一下基本概念。

我们知道两个进程如果需要进程通讯，最基本的前提就是保证彼此进程的唯一，并能确定彼此身份。在本地进程通讯中我们可以使用 PID 来标示唯一的进程，但 PID 只在本地唯一，网络中的两个进程 PID 冲突的几率很大，这时候我们就需要另辟蹊径了。

我们知道IP 层的IP 地址可以唯一标示主机，而TCP 层协议和端口号可以唯一标示主机的一个进程，这样我们就可以利用 IP 地址+ 协议 + 端口号唯一标示网络中的一个进程。

能够唯一标示网络中的进程后，它们就可以利用socket 进行通信了。

> 什么是socket？

我们经常把 socket 翻译成套接字，**socket 是在应用层和传输层之间的一个抽象层，它把 TCP/IP 层复杂的操作抽象为几个简单的接口供应用层调用以实现进程在网络中通信**。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201015094303.png)

socket 起源于 UNIX，在UNIX 一切皆为文件哲学的思想下，socket 是一种“打开=>读/写=>关闭“模式的实现，服务器和客户端各自维护一个文件，在建立连接打开之后，可以向自己的文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。

### socket 通信流程
socket 是"打开=>读/写=>关闭"模式的实现，以使用TCP协议通讯的socket为例，其交互流程大概是这样子：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201015094756.png)

## 安装
PHP 默认没有启用 sockets 扩展，所以需要手动安装扩展。

```
apt-get install php7.2-sockets
```

`php -m` 或者 `php -i`检查扩展是否已经启用。

### 创建连接

创建并返回一个套接字，也称作一个通讯节点。一个典型的网络连接由 2 个套接字构成，一个运行在客户端，另一个运行在服务端。

```
<?php
# 创建一个TCP 协议的 socket
$socket = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);

# 创建一个本地的socket
$socket = socket_create(AF_UNIX, SOCK_STREAM, 0);
```
`socket_create`函数接收三个参数，分别是domain、type、protocol。 
* domain：当前套接字使用什么协议
* type：当前套接字的类型
* protocol：设置指定 domain 套接字下的具体协议

### 发送内容

发送数据有两种方式：
1. socket_send：发送消息至已连接的客户端。
2. socket_sendto：发送消息至客户端，无论是否连接。
```
<?php
$sock = socket_create(AF_UNIX, SOCK_DGRAM, SOL_UDP);

$msg = "Ping !";
$len = strlen($msg);

// 向本地 1223 端口发送内容
socket_sendto($sock, $msg, $len, 0, '127.0.0.1', 1223);
socket_close($sock);
```

### 接收数据
接收数据也有两种方式：
1. socket_recv：从已连接的socket 接收数据
2. socket_recvfrom：从socket 接收数据，无论是否连接

```
<?php
$sock = socket_create(AF_UNIX, SOCK_DGRAM, SOL_UDP);

# 从本地 1223 端口获取内容
socket_recvfrom($socket, $buf, 1024, 0, "127.0.0.1", 1223);
var_dump($buf); // Ping !
```

### 参考链接
* [简单理解Socket](https://www.cnblogs.com/dolphinx/p/3460545.html)
* [一篇搞懂TCP、HTTP、Socket、Socket连接池](https://segmentfault.com/a/1190000014044351)
* [socket_create](https://www.php.net/manual/zh/function.socket-create.php)
* [socket_sendto](https://www.php.net/manual/zh/function.socket-sendto.php)
* [socket_bind](https://www.php.net/manual/zh/function.socket-bind.php)
