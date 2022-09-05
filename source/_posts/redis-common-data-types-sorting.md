---
title: Redis 常用数据类型整理
date: 2020-11-07 22:43:21
tags: ["Redis"]
categories: ["Redis"]
---

Redis 的五种数据类型分别是：字符串、哈希、列表、集合、有序集合。

<!-- more -->

## string
String 是Redis 最基本的数据类型，一个 Key 对应一个 Value。

String 类型是二进制安全的。意思是 Redis 的 String 可以包含任何数据。（数字：整浮型点数，二进制：图片、音频、视频、序列化的对象）

String 类型是 Redis 最基本的数据类型，一个键最大能存储 512 MB。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107122134.png)

### 应用场景
* `incr`：计数
* `set` + `get`：将对象/Json 序列化之后存储作为Cache

### 快速上手
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220905134752.png)

## hash
Redis hash 是一个键值对集合。

Redis hash 是一个 String 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107123137.png)

### 应用场景
* `hset` + `hget`：Cache

### 快速上手
在下面的例子中，“rediscomcn” 是 Redis 哈希，它包含详细信息（name，url，rank，visitors）属性。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220905134804.png)

## list
用来存储多个有序的字符串，一个列表最多可以存 2 的 32 次方减 1 个元素。

列表的特点是：
1. 有序
2. 允许重复

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107122342.png)

### 应用场景
* `lpush` + `lpop`：Stack
* `lpush` + `rpop`：Queue
* `lpush` + `ltrim`：Capped Collection
* `lpush` + `brpop`：Message Queue

### 快速上手

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220905134813.png)

## set
集合特点：
1. 无序
2. 不允许重复

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107122610.png)

### 应用场景
* `sadd`：Tagging
* `spop/srandmember`：Random item
* `add` + `sinter`：Social Graph

### 快速上手
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220905134823.png)

## sorted set

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107123412.png)

### 应用场景
* `zscore`：timeStamp、saleCount、followCount

### 快速上手

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220905134845.png)

## 列表、集合、有序集合的区别

|数据结构|是否允许元素重| 是否有序| 有序实现方式|应用场景|
| ------- | ------- | ------- | ------- | ------- |
| 列表 | 是 | 是 | 索引下标 | 时间轴，消息队列 | 
| 集合 | 否 | 否 | 无 | 标签，社交 |
| 有序集合 | 否 | 是 | 分值 | 排行榜，点赞数 |


## 通用命令

查看所有key：
```
keys *
```

查看加载配置文件：
```
config get * 
```

当前数据库的 key 的数量：
```
dbsize
```

判断key 是否存在：
```
exists key
```

删除key：
```
del key
```

查看key 的类型：
```
type key
```

查看内存使用情况：
```
info memory
```