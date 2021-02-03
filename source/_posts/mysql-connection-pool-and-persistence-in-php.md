---
title: mysql connection pool and persistence in php
date: 2021-02-03 19:42:17
tags: PHP 中实现 Mysql 连接池与持久化
---

Mysql 的连接方式有两种：tcp 和 socket。前者是基于`tcp/ip`协议，后者是基于socket 套接字。

<!-- more -->

具体：
* tcp/ip：`mysql -h 127.0.0.1 -uroot -p`
* socket：`mysql -h localhost -uroot -p` 或者 `mysql -uroot -p`

可以通过 `tcpdump`命令抓包。

指定源端口：
```
$ tcpdump -i lo0 port 3306
```

如果出现以下内容，表示本地没有`lo0`这个设备。
```
tcpdump: lo: No such device exists
(BIOCSETIF failed: Device not configured)
```

可以通过`tcpdump -D` 命令查看本地设备名称：
```
1.en0 [Up, Running]
2.lo0 [Up, Running, Loopback]
```

使用`mysql -h 127.0.0.1 -uroot -p`，可以看到Mysql 的连接过程是基于`tcp/ip` 协议。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210118172506.png)

当客户端退出Mysql 时，会发送四条记录，也就是tcp 的四次挥手。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210118172203.png)

`socket` 方式会快于`tcp/ip`，

mysql 使用线程来处理连接，每当一个请求进来，MySQL会创建一个线程去处理请求，

可以使用`show status`命令查看当前处于连接状态的线程个数，所以在高并发下，这将给MySQL服务器带来巨大的压力，消耗服务器资源。

### Mysql 线程池
实际上 mysql 实现了线程池，当客户端断开连接后，mysql 会将当前线程缓存起来，当下一次有新的请求进来时，无需创建新的线程。

查看线程池大小：
```
mysql> show variables like 'thread_cache_size';
```

设置线程池大小：
```
mysql> set global thread_cache_size = 20;
```

查看线程池状态：
```
mysql> show status like 'Threads_%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 8     |
| Threads_connected | 3     |
| Threads_created   | 53    |
| Threads_running   | 1     |
+-------------------+-------+
4 rows in set (0.02 sec)
```
其中：

* `Threads_cached`：空闲线程数量。当有新的请求进来时，mysql 不会立即创建线程去处理，而是去`Threads_cached`查看空闲的连接线程，如果存在则直接使用，不存在则创建新的线程。
* `Threads_connected`：当前处于连接状态的线程个数。
* `Threads_created`：创建过的线程数，如果发现`Threads_created`值过大的话，表明 mysql 服务器一直在创建线程，这也是比较耗资源，可以适当增加配置文件中`Thread_cache_size`值。
* `Threads_running`：处于激活状态的线程的个数，这个一般都是远小于`Threads_connected`的。

线程池的出现解决了频繁的创建连接和销毁连接的问题，但仅有线程池还是不够的，不能解决客户端频繁连接mysql 带来的性能损耗。

### 参考链接
* [PHP中实现MySQL连接池与持久化](https://www.wugenglong.com/post/mysql_connection_pool/)
* [用Swoole4 打造高并发的PHP协程Mysql连接池](https://my.oschina.net/u/2394701/blog/2046414)