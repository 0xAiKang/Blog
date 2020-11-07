---
title: Redis 常用数据类型整理
date: 2020-11-07 22:43:21
tags: ["Redis"]
categories: ["Redis"]
---

Redis 的五种数据类型分别是：字符串、哈希、列表、集合、无序集合。

<!-- more -->

## Redis 的五种数据类型

### string
字符串是Redis 的五种数据类型中，最常见最好理解的。

它的数据结构最简单，就是一个标准的键值对，一个key 对应一个value：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107122134.png)

key 是string 的键，可以是字符串也可以是数字。
value 是string key 所对应的值，可以是字符串也可以是数字。

#### 应用场景
* `incr`：计数
* `set` + `get`：将对象/Json 序列化之后存储作为Cache

### hash
哈希的数据结构很像一个迷你的关系数据库。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107123137.png)

#### 应用场景
* `hset` + `hget`：Cache

### list
列表的特点是：
1. 有序
2. 允许重复

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107122342.png)

#### 应用场景
* `lpush` + `lpop`：Stack
* `lpush` + `rpop`：Queue
* `lpush` + `ltrim`：Capped Collection
* `lpush` + `brpop`：Message Queue

### set
集合特点：
1. 无序
2. 不允许重复

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107122610.png)

#### 应用场景
* `sadd`：Tagging
* `spop/srandmember`：Random item
* `add` + `sinter`：Social Graph

### sorted set

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201107123412.png)

#### 应用场景
* `zscore`：timeStamp、saleCount、followCount

### 通用命令

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