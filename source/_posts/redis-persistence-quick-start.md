---
title: Redis 持久化快速上手
date: 2020-11-21 19:05:31
tags: ["Redis","持久化"]
categories: ["Redis"]
---

> 什么是持久化？

Redis 所有数据都是存储在内存中的，对于数据的更新将异步的保存在磁盘中，当Redis实例重启时，即可利用之前持久化的文件实现数据恢复。

主流数据库的持久化方式：
* 快照
  * Mysql dump
  * Redis rdb
* 日志
  * Mysql binlog
  * Redis aof

## RDB
### 什么是RDB？
Redis 通过一条命令或者某种方式创建 rdb 文件，该文件是二进制格式，存储在硬盘中。

当需要对Redis 进行恢复时，就可以去加载该文件。

数据恢复的程度，取决于 rdb文件（快照）产生的时刻。

### 三种触发机制
Redis 生成 rdb 文件有三种方式，分别是：
* save
* bgsave
* 自动策略

#### save 
save 命令有如下特点：
1. 同步阻塞
2. 文件策略：如果存在旧的rdb 文件，则会替换成新的
3. 复杂度：O（N）

#### bgsave
bgsave 命令有如下特点：
* 异步非阻塞（几乎不会阻塞客户端）
* 文件策略和复杂度同上。

> save 还是 bgsave？

|命令|save|bgsave|
|-|-|-|
|IO类型|同步|异步|
|是否阻塞|是|否（阻塞发生在fork()|
|复杂度|O(n)|O(n)|
|优点|不会消耗额外内存|不阻塞客户端|
|缺点|阻塞客户端|需要fork，消耗内存|

在数据量不大的情况下，其实使用save 还是bgsave 并没有什么差异。

#### 自动策略
自动生成策略是根据某个规则来决定是否生成 rdb 文件，这个过程也是一个bgsave 的过程。

默认策略：
|seconds|changes|
|-|-|
|900|1|
|300|10|
|60|10000|

上述配置的意思是：如果在60s 中做了10000 次改变或者在 300s 中做了 10次 改变，或者在900s 中做了 1 次改变，则均会触发bgsave。

#### 配置
```
#save 900 1
#save 300 10 
#save 60 10000
dbfilename dump-${port}.rdb       // rdb 文件名称
dir /big_disk_path                // 工作目录
stop-writes-on-bgsave-error yes   // 如果发生错误，停止写入
rdbcompression yes                // 采用压缩格式 
rdbchecksum yes                   // 对rdb 文件进行检验
```
### 触发机制
Redis 当达到以下触发机制时，就会自动创建rdb 文件。
1. 全量复制
2. debug reload
3. showdown

## AOF
在正式介绍什么是AOF 之前，我们先来了解一下RDB 方式现存的问题。
1. 耗时、耗性能
2. 不可控、丢失数据

### 什么是AOF？
Redis 的每次操作都会产生日志，AOF 文件可以理解成记录操作内容。
AOF 其实就是根据Redis 的执行日志去创建AOF 文件。

### 三种策略
Redis 在执行写命令时，首先写入硬盘的缓冲区，缓冲区会根据以下策略去刷新到磁盘中。
* always：每条命令 fsync 到硬盘。
* everysec：每秒把缓冲区 fsync 到硬盘。
* no：由系统决定是否 fsync。

> always 还是 everysec 还是 no？

|命令|always|everysec|no|
|-|-|-|-|
|优点|不丢失数据|每秒一次 fsync |不用管|
|缺点|IO 开销较大，一般的sata 盘只有几百 TPS|丢一秒数据|不可控|

#### 配置
```
appendonly yes                              // 开启AOF 策略
appendfilename "appendonly-${port}.aof"     // aof 文件名
appendfsync everysec                        // 刷新策略
dir /big_disk_path                          // 工作目录
no-appendfsync-on-write  yes                // AOF 重写时，是否需要做AOF 正常操作
auto-aof-rewrite-percentage 100             // 重写增长率
auto-aof-rewrite-min-size 64mb              // 重写最小存储
```

### 总结
关于究竟选择RDB 还是AOF，并没有绝对正确的答案。需要根据实际情况去作取舍。

|命令|RDB|AOF|
|-|-|-|
|启动优先级|低|高|
|体积|小|大|
|恢复速度|快|慢|
|数据安全性|丢数据|根据策略决定|
|级别|重|轻|