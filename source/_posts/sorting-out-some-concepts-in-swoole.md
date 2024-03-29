---
title: Swoole 常见概念整理
date: 2020-10-25 20:24:52
tags: ["PHP", "Swoole"]
categories: ["PHP"]
---

[Swoole](https://www.swoole.com/) 是一个非常优秀的PHP 的协程高性能网络通信引擎。

在学习过程中，遇到了一些新或旧的概念，在此整理一下。

<!-- more -->

## 长连接/短连接
长连接： 客户端和服务端建立连接后不进行断开，之后客户端再次访问这个服务器上的内容时，继续使用这一条连接通道。
短连接： 客户端和服务端建立连接，发送完数据后立马断开连接。下次要取数据，需要再次建立连接。

## 串行/并行/并发
串行：执行多个任务时，各个任务按顺序执行，完成一个之后才能进行下一个。\
并行：多个任务在同一时间点发生并执行。\
并发：同一时间段需要执行多个任务

## IO（Input/Output，输入输出）
在计算机中，输入 / 输出（即 IO）是指信息处理系统（比如计算机）和外部世界（可以是人或其他信息处理系统）的通信。

输入是指系统接收的信号或数据，输出是指从系统发出的数据或信号。

涉及到IO 操作的通常有磁盘、网络、文件等。

## 同步/异步
**同步和异步是一种消息通信机制**。其关注点在于 `被调用者返回` 和 `结果返回` 之间的关系， 描述对象是被调用对象的行为。

同步：在发出一个同步调用后，没有得到结果返回之前，该调用就不会返回，只有等待结果返回之后才会继续执行后续操作。
异步：发出调用，直接返回。异步可以通过状态、回调、通知调用者结果，可以先执行其他操作，直到回调结果返回之后，再回来执行回调那部分的操作。

## 阻塞/非阻塞
**阻塞和非阻塞是一种业务流程处理方式**。关注点在于调用发生时 `调用者状态` 和 `被调用者返回结果` 之间的关系。 

描述的是等待结果时候调用者的状态。 此时结果可能是同步返回的，也能是异步返回。

阻塞：在结果返回之前，该线程会被挂起，后续代码只有在结果返回后才能执行。
非阻塞：在不能立刻获取结果前，该调用不会阻塞当前线程。

## 同步阻塞/非同步阻塞
实际编程中，通过**线程**实现**进程**的**同步非阻塞**，通过**协程**实现**线程**的**同步非阻塞**。

同步阻塞：打电话问老板有没有某书（调用），老板说查一下，让你别挂电话（同步），你一直等待老板给你结果，什么事也不做（阻塞）。

同步非阻塞：打电话问老板有没有某书（调用），老板说查一下，让你别挂电话（同步），等电话的过程中你还一边嗑瓜子（非阻塞）。

## 异步阻塞/异步非阻塞

异步阻塞：打电话问老板有没有某书（调用），老板说你先挂电话，有了结果通知你（异步），你挂了电话后（结束调用）, 除了等老板电话通知结果，什么事情也不做（阻塞）。

异步非阻塞：打电话问老板有没有某书（调用），老板说你先挂电话，有了结果通知你（异步），你挂电话后（结束调用），一遍等电话，一遍嗑瓜子。（非阻塞）

### 参考链接
* [Swoole 中涉及的一些基本概念](https://learnku.com/articles/45280)
