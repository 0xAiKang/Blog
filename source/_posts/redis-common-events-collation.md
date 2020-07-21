---
title: Redis 常见事件整理
date: 2020-07-21 23:41:45
tags: ["Redis"]
categories: ["Redis"]
---

这篇笔记用来整理 Redis 的常用事件。

<!-- more -->

## 客户端事件
客户端会发出一些事件的状态连接到Redis 服务器。

### Ready

### Error
客户端连接Redis 时，如果出现异常，则会触发Error 事件。

### Connect
客户端连接至Redis 时，会触发连接事件。

## 订阅者事件
### Message
将接收到来自订阅频道的消息，

```
client.on("message", function (channel, message) {
    ...
})
```
### Subscribe
监听订阅事件，返回订阅频道的订阅数量。

```
client.on("subscribe", function (channel, count) {
    ...
})
```

## 发布/订阅

### Publish

将信息 `message` 发送到指定的频道 `channel` 。

返回值：接收到信息 `message` 的订阅者数量。
```
PUBLISH channel message
```

### SUBSCRIBE

订阅给定频道的信息。

返回值：接收到的信息。
```
SUBSCRIBE channel [channel ...]
```

### 参考链接
* [Redis命令参考简体中文版](https://redis.readthedocs.io/en/2.4/index.html)
* [A high performance Node.js Redis client](https://github.com/NodeRedis/node-redis)
