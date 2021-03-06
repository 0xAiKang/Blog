---
title: PHP 常见面试题整理
date: 2021-02-25 19:51:45
tags: ["PHP", "面试"]
categories: ["PHP"]
---

年前年后这段时间一直在为面试做准备，本文将从网络、PHP、Mysql、Redis、Linux 这几部分整理一下常见的一些面试简答题。

<!-- more -->

## 网络篇

### 常见的状态码及其含义

|状态码|含义|
|-|-|
|200|请求成功|
|301|重定向|
|304|资源未被修改可以使用旧资源|
|404|资源找不到|
|403|请求被拒绝|
|500|服务端错误|
|502|网关错误|
|504|网关超时|

### 表单提交 get 和 post 的区别
1. Get 请求是将请求参数放在 url 后面，等同于直接放在了请求头中，Post 请求则是把请求参数放在请求体中。
2. Post 更安全，不会作为url的一部分，不会被缓存及保存在浏览器记录中。
3. Post 能发送更多的内容及更多的数据类型，get 只能发送 2048 个ASCII 字符
4. Post 比Get 慢（原因是因为post 需要在服务器确认之后再发送数据）
5. Get 通常用于资源的获取，Post 通常用于资源的更新

### http 和 https 的区别
1. 首先两者所使用的协议不一样，其端口也不一样。
2. 这也是https 比 http 要安全的原因，http 是明文传输，数据都是未加密的，而https 则是 ssl + http 协议进行加密传输。
3. http 比 https 要快，这是因为http 只需要进行tcp 三次握手连接，只需要交换三个包，而 https 除了进行tcp 连接，还需要 ssl 握手的九个包，一共是十二个包。
4. https 是构建在http 之上的协议，理论上，https 相较 http 会更消耗服务器资源。

### session 和 cookie 的区别
1. 存储方式不同：cookie 是存储在客户端，session 则是存储在服务端。
2. 隐私策略不同：cookie 因为存储在客户端中，所以对客户端是可见的，而session 

### UDP 和TCP 的区别
UDP 是面向报文的、不可靠的数据报协议，TCP 是面向连接的、可靠的流协议。

1. TCP 面向连接; UDP 不需要连接，即发送数据之前不需要建立连接;
2. TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达; UDP尽最大努力交付，即不保保证可靠交付
3. TCP面向字节流，实际上是TCP把数据看成一连串无结构的字节流; UDP是面向报文的，UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低
4. 每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信
5. TCP首部开销20字节;UDP的首部开销小，只有8个字节
6. TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道

> 说一说TCP 的“粘包”问题

结论：TCP 的“粘包”问题其实是一个伪命题。

服务端建立服务，客户端发起连接，正常情况下，服务端每次send，客户端都能正常recv，但在并发的情况下，服务端的两次send或者多次send，客户端可能只有一次recv了。这就导致了所谓的“粘包”问题的产生。

TCP 协议的本质是流协议，它只会保证以什么顺序发送字节，接受方就一定能按照这个顺序接收到，所以所谓的粘包问题不应该是传输层面的问题，而是应用层面的问题。

### 简述TCP 三次握手
概念：指在发送数据的准备阶段，服务器和客户端之间需要三次交互。

第一次握手：客户端向服务端发送一个SYN包，并进入SYN_SENT 状态，等待服务端确认
第二次握手：当服务器收到请求之后，此时要给客户端一个确认信息ACK，同时发送SYN报，此时服务器进入 SYN_RECV 状态
第三次握手：客户端收到服务器发的ACK + SYN 包后，向服务器发送ACK，发送完毕之后，客户端和服务器进入TCP 连接成功状态，完成三次握手。

> 为什么握手一定要三次，不能两次吗？

这是为了防止已经失效的连接请求报文突然又传到Tcp 服务器，避免产生错误。

### 简述TCP 四次挥手
概念：所谓四次挥手就是说关闭TCP 连接的过程，当断开一个TCP 连接时，需要客户端和服务器共共发送四个包确认。

第一次挥手：客户端发送一个FIN，用来关闭客户端到服务器的数据传输，客户端进入 fin_wait 状态
第二次挥手：服务器收到fin 后，发送一个ack 给客户端，确认序号为收到序号+1，服务器进入close_wait 状态
第三次挥手：服务器发送一个fin 用来关闭服务器到客户端的数据传输，服务器进入 last_ack 状态
第四次挥手：客户端收到fin 后，客户端进入time_wait 状态，接着发送一个ack 给服务器，确认序号为收到序号+1，服务器进入 closed 状态，完成四次挥手。

### 建立socket 需要哪些步骤
* 创建socket
* 绑定socket 到指定地址和端口
* 开始监听连接
* 读取客户端输入
* 关闭 socket

### 简述从浏览器输入 a.com 回车之后发生了什么
1. DNS 域名解析，寻找对应的IP 地址
2. 根据这个IP 找到对应的服务器，建立TCP 连接（三次握手）
3. TCP 建连之后，发起HTTP 请求
4. 服务器响应 HTTP 请求
5. 客户端接收数据解析并渲染页面
6. 服务器关闭TCP 连接（四次挥手）

### 长连接与短连接的区别
短连接为每一次的数据传输准备了一个传输通道，而长连接则是建立一条通道，并一直保持，每一次传输时都会复用同一条连接通道。

### Websocket 
Websocket 是一种通信协议，连接刚开始还是HTTP 协议，由客户端发起，然后切换成Websocket 协议。

它的存在，由轮询变成了客户端可以主动向服务端发送消息。

> 什么是轮询？

轮询是一种获取信息的方式。

## PHP篇

### 值传递和引用传递的区别
值传递：传递的是变量在内存中的副本。
引用传递：传递的是变量在内存中的地址。

unset 并不会真正意义上注销一个变量，而是切断了变量名和实际值之间的关系，其变量只要还被引用就还没有被释放。

### composer 自动加载原理
composer 的核心加载思想是通过composer 的`autoload.php`，将类和路径的对应关系加载到内存中，最后将具体的加载实现注册到 `spl_autoload_register` 函数中。

### 常用的请求第三方接口有哪几种方式？
1. `curl`
2. `file_get_contents`
3. `fopen`

### 抽象类和接口类的区别
抽象用于描述不同的事物，接口用于描述事物的行为

### 进程与线程的区别
进程是CPU 分配内存的最小单位，线程是CPU 调度的最小单位，一个进程可以有多个线程，一个线程只能有一个进程。

### Swoole 的进程模型
Swoole 的进程模型采用主进程、管理进程、异步任务/工作进程协作的方式。

* Manager 进程主要负责创建/回收 worker/task 进程
* Reactor 进程主要负责维护客户端 TCP 连接、处理网络 IO、处理协议、收发数据
* Worker/Task 进程主要负责执行PHP 代码。

### PHP 的进程模型
在LNMP 的模式下，PHP 是php-fpm 多进程+阻塞I/O 的进程模型。

### 同步、异步、阻塞、非阻塞是怎么回事？
* 同步和异步是一种消息通信机制。
* 阻塞和非阻塞是一种业务流程处理方式。
* IO 多路复用：用一个线程来检查Socket 的就绪状态。

### 并发、并行有什么区别？
并发：两件或者多件事情在同一时间间隔内发生
并行：两件或者多件事情在同一时刻发生

区别在与：在同一个时刻，对于并行来说，事件是一并发生，而对于并发来说，在宏观看来也是一并发生，但在微观上却是交替发生。

### 简述PHP 代码解析过程
Zend 引擎首先会将PHP 代码进行解析（词法、语法解析）成 opcode，然后Zend 虚拟机会顺序执行这些指令。

### 从LNMP 的角度简述一次完整的请求过程
当客户端发起一个请求到服务端，Web Server 首先会判断该请求是静态还是动态？
如果是静态，直接返回对应的静态资源。
如果是动态，FastCGI 会将该请求转发给本地 9000 端口（9000 是 PHP—FPM 所监听的端口），PHP-FPM 主进程接收到请求之后，
会分配一个空闲的 Worker 进程去处理这个请求，处理完成之后将数据返回给 FastCGI，再由 Nginx 返回给客户端。

### PHP 可以做常驻内存吗？为什么？
传统的PHP 无法以常驻内存的方式运行，这是因为PHP是解释型脚本语言，这种运行机制使得每个PHP 页面解释执行后，所有资源都被回收了。

### 通常如何实现用户登录（API）
有两种方式：一种是普通的 token 令牌，另一种则是JWT。

普通Token：
用户初次登录会携带用户名和密码等信息，服务端验证通过之后，会给客户端返回一个Token。
这个Token 可以是由用户名、密码、登录设备、登录IP 等信息加密之后组成，
以后的客户端每一次请求都会携带这个Token，服务端则会验证该Token。

### 在PHP中，你是如何捕获异常的？
尽量避免使用`exit`、`die`方法直接退出，而使用`try...catch`来捕获异常。

### 常见设计模式
创造型：工厂模式、单例模式、原型模式
结构型：适配器模式、装饰模式、门面模式、代理模式
行为型：迭代器模式、中介模式、观察者模式

### 什么是依赖注入
依赖注入主要用来减少代码间的耦合，有效分离对象和它所需要的外部资源。

## Mysql篇
Mysql 的InnoDb 和MyISAM 引擎有何不同？
* InnoDb 的特点包括：事务、锁
* InnoDb 支持 ACID 的事务 4个特性，MyISAM 不支持事务。
* InnoDB 支持行级别的锁粒度，MyISAM 不支持，只支持表级别的锁粒度。

### 什么是ACID（事务的四个特性）？
* 原子性（Atomicity）：事务的所有操作，要么全部完成，要么全部不完成，不会结束在某个中间环节。
* 一致性（Consistency）：事务开始之前和事务结束之后，数据库的完整性限制未被破坏。
* 隔离性（Isolation）：两个或者多个事务的执行是互不干扰的，一个事务不可能看到其他事务运行时，中间某一时刻的数据。
* 持久性（Durability）：事务完成之后，事务所做的修改进行持久化保存，不会丢失。

### Mysql 有几种事务隔离级别？
有四种隔离级别。

### 死锁是什么？
两个或多个事务在同一个资源上相互占用。

### 简述Mysql 索引、主键及其常见索引
索引就是类似于书籍目录的存在，主键是用于确定字段的唯一性。

* 普通索引：最普通的索引，使用没有什么限制。
* 唯一索引：与普通索引类型，唯一不同的是，列值不允许重复，但允许有空值。
* 主键索引：主键本身自带的索引，不允许有空值。
* 全文索引：仅可用于 MyISAM 表，针对较大的数据，生成全文索引很耗时占空间
* 组合索引：为了提高多条件查询效率，可建立组合索引，遵循”最左前缀匹配原则"

但是索引也不是越多越好，索引加快了查询速度，但同时也会影响更新速度。

## Redis篇
### Redis 和Memcache 的区别
* Redis 支持多种数据类型，Memcache 只支持Key-Value
* Redis 支持两种持久化，Memcache 不支持持久化。

### Redis 的常见数据结构及其应用场景
* 字符串
* 哈希
* 列表
* 无序列表
* 集合

### Redis 的持久化有几种方案？
有三种，分别是：RDB、AOF、混合。

1. RDB：将某一时刻的数据以二进制形式写入到磁盘里，服务重启时检测到对应文件自动加载进行数据恢复，有手动触发和自动触发两种机制。
2. AOF：以文件追加的方式写入客户端执行的写命令，数据恢复时，通过创建伪客户端的方式执行命令，直到恢复完成。
3. 混合：在写入的时候先把数据以 RDB 的形式写入文件的开头，再将后续的写命令以 AOF 的格式追加到文件中。

### 为什么Redis 是单线程？

## Linux篇
### 说一下你常用的Linux 命令（最基础的不用说）

* 网络：`ping`、`tcpping`、`telnet`、`netstat`、`nmap`、`lsof`、`tcpdump`
* 磁盘：`df`、`du`
* 进程：`ps`、`pstree`
* 内存：`free`
* 负载：`top`
* 压测工具：`ab`、`wrk`
* 文件上传/下载：`curl`、`wget`、`scp`
* 综合：`glances`

### 基本的运维需要监控哪些数据？
* 系统层：CPU、内存、负责、网卡、I/O
* 应用层：QPS、API响应时长、Redis内存使用量、任务队列数、php-fpm 进程数、Mysql线程数
* 健康巡查：dns 解析、ip 是否可以访问、硬盘、各种基础服务

## 其他

### 说一说你所知道的网站攻击方式及如何防范
* CSRF 跨站伪造请求
* XSS 跨站脚本攻击
* SQL 注入
* DDOS 攻击

### 如果用户反馈网站慢，你会怎样做？
1. 资源加载慢
  i. WebServer 配置静态资源缓存、动静分离
  ii. DNS 缓存、CDN 加速
  iii. 增加服务器带宽
2. SQL 查询慢
  i. Mysql 慢查询找出耗时SQL
  ii. Explain 分析耗时原因
  iii. 优化SQL
3. 并发
  i. 优化PHP-FPM 配置

### 如果你发现你部署的网站打不开了，你会如何排查？
1. 首先检查DNS 解析
  i. 检查域名解析
  ii. 排除浏览器缓存
2. 检查防火墙
  i. 防火墙是否开启？
  ii. 端口是否可以正常访问，通常使用telnet 命令检查
3. 根据网站返回状态码，具体分析
  i. 404：访问资源不存在
  ii. 500：服务端错误（代码错误、文件权限）
  iii. 502：网关错误（webserver 异常导致，Nginx/Apache 发生错误）
  iiii. 504：网关超时
4. 查看对应的日志

### 参考链接
* [PHP面试问答](https://github.com/colinlet/PHP-Interview-QA)