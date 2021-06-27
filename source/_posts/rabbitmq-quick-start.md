---
title: RabbitMQ 快速上手
date: 2021-06-27 18:34:43
tags: ["MQ", "RabbitMQ"]
categories: ["MQ"]
---

RabbitMQ 学习笔记整理。

<!-- more -->

## Rabbit 是什么？
它是一个开源的**消息代理和队列服务器**，用来通过普通协议在完全不同的应用中共享数据，Rabbit 是使用Erlang 语言编写的，而RabbitMQ 是基于 AMQP 协议的。

> 如果你还不知道消息队列是什么，可以查阅我的这篇笔记——[快速上手消息队列](https://www.0x2beace.com/quick-start-message-queue/#%E4%BB%80%E4%B9%88%E6%98%AF%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%EF%BC%9F)

### 成熟的消息队列那么多，为什么要选择RabbitMQ？

工具的选择往往并不直接取决于该工具本身多么优秀，而是该工具能提供的功能，刚才符合我们的需求。

RabbitMQ 也是，在众多成熟的消息队列中，它的特点如下：
1. 开源、性能优秀
2. 提供可靠性消息投递模式、返回模式
3. 集群模式丰富，支持表达式配置，HA 模式，支持镜像队列模型
4. 保证数据不丢失的前提下做到高可靠性、可用性

### 什么是AMQP 高级协议？
AMQP 全称（Advanced Message Queuing Protocol）高级消息队列协议。

它是一个具有现代特征的二进制协议，是一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向信息的中间件设计。

这个解释太官方了，说了跟没说似的。

其实呢，通俗一点讲，它就是一个规范，约定了一个**核心概念**，开发时需要遵守该规范。

### AMQP 核心概念
AMQP 核心概念由以下部分组成：
* `Server`：又称 Broker，接受客户端连接，实现AMQP 实体服务
* `Connection`：连接，应用程序与Broker 的网络连接
* `Channel`：网络信道，几乎所有的操作都在Channel 中进行，Channel 是进行消息读写的通道，客户端可建立多个Channel，每个Channel 代表一个会话任务。
* `Message`：消息，服务器与应用程序之间传送的消息，由Properties 和Body 组成，Properties 可以对消息进行修饰，比如消息的优先级、延迟等高级特性;  Body 就是消息体内容。
* `Virtual host`：虚拟主机，用于进行逻辑隔离，最上层的消息路由，一个Virtual host 里面可以有若干个 Exchange 和Queue，同一个Virtual host 里面不能有相同名称的Exchange 和 Queue 。
* `Exchange`：交换机，接收消息，根据路由键转发消息到绑定的队列
* `Binding`：Exchange 和Queue 之间的虚拟连接，binding 中可以包含 routing key
* `Routing key`：一个路由规则，虚拟机可以用他来确定如何路由一个特定消息
* `Queue`：也称为Message Queue，消息队列，保存消息并将它们转发给消费者

## RabbitMQ 整体架构
RabbitMQ 整体架构，可以抽象看成三部分组成：
1. Produce：消息生产者
2. Service：AMQP 服务
3. Consume：消息消费者

一个简单的RabbitMQ 架构图：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210627160843.png)

在上图中，生产者首先把消息投递到Server 的Exchange 中，然后Exchange 将消息流转到某个Queue 中，生产者监听指定队列从中获取消息。

在该流程中，生产者不需要关心，将消息投递到哪个队列，只需要关心，将消息投递到哪个Exchange。
消费者也不需要关心，消息是从哪个Exchange 中获取的，只需要关心，监听哪个队列。

> 那么Exchange 与Queue 之间又是如何进行消息流转的呢？

虽然一个Exchange 可以绑定多个Queue，但是路由策略（Routing Key）决定了，最终将消息投递到哪个具体Queue上。

## RabbitMQ 快速上手
和其他消息队列一样，想要使用，需要先安装队列服务器并启用。

具体安装过程就不过多介绍，可以前往[官网](https://www.rabbitmq.com/download.html)查看对应操作系统的安装流程。

安装完成之后，通过`rabbitmq-server start_app` 命令，启动RabbitMQ 服务。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210627163241.png)

如果能看到以上输出，则表示服务已正常启动。

### 常见概念
* 5672：通信端口号
* 15672：管控台端口号
* 25672：集群端口号
* rabbitmqctl：基础服务管理
* rabbitmq-plugins：插件管理
* rabbitmq-server

访问 `127.0.0.1:15672` 即可看到RabbitMQ 的控制台，默认的账号密码分别为：`guest`、`guest`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210627163710.png)

### 常用命令

#### 服务器管理
启动应用：
```bash
rabbitmqctl start_app
```

关闭应用：
```bash
rabbitmqctl start_app
```

节点状态：
```bash
rabbitmqctl status
```

#### 用户管理
添加用户：
```bash
rabbitmqctl add_user username password
```

列出用户：
```bash
rabbitmqctl list_user
```

删除用户：
```bash
rabbitmqctl delete_user username
```

清除用户权限：
```bash
rabbitmqctl clear_permissions -p vhostpath username
```

列出用户权限：
```bash
rabbitmqctl list_user_permissions username
```

修改密码：
```bash
rabbitmqctl change_password username newpassword
```

设置用户权限：
```bash
rabbitmqctl set_permissions -p vhostpath username
```

#### 虚拟主机管理
创建虚拟主机
```bash
rabbitmqctl add_vhost vhostpath
```

列出虚拟主机
```bash
rabbitmqctl list_vhosts
```

列出虚拟主机的所有权限：
```bash
rabbitmqctl list_permissions -p vhostpath
```

删除虚拟主机：
```bash
rabbitmqctl delete_vhost vhostpath
```

#### 队列管理
查看所有队列信息：
```bash
rabbitmqctl list_queues
```

清除队列里的信息：
```bash
rabbitmqctl -p vhostpath purge_queue blue
```

移除所有数据，要在rabbitmqctl stop_app 之后使用：
```bash
rabbitmqctl reset
```
#### 节点管理
修改集群节点的存储形式：
```bash
rabbitmqctl change_cluster_node_type disc|ram
```

忘记节点（摘除节点）：
```bash
rabbitmqctl forget_cluster_node [--offline]
```

修改节点名称：
```bash
rabbitmqctl rename_cluster_node oldnode1 newnode1
```