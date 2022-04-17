---
title: Mysql Too many connections 异常记录
date: 2022-04-17 11:16:50
tags: ["Mysql"]
categories: ["Mysql"]
---

前段时间，线上业务偶尔会出现 `SQLSTATE[HY000] [1040] Too many connections` 的异常。

通常是以下两种原因之一造成的：
 1. `max_connections` 配置过小
 2. 访问量过高

 <!-- more -->

## 知识点

### max_connections

Mysql 的 [max_connections](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_connections) 是限制允许客户端同时连接的最大连接数。

```mysql
mysql> show variables like '%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 500   |
+-----------------+-------+
1 row in set (0.01 sec)
```

Mysql 无论如何都会保留一个用于管理员（SUPER）登陆的连接，用于管理员连接数据库进行维护操作，所以即便当前连接数已经达到了 `max_connections`，管理员仍可以连接，因此 Mysql 的实际最大可连接数为 `max_connections+1`。

增加 `max_connections` 参数的值，不会占用太多系统资源。系统资源（CPU、内存）的占用主要取决于查询的密度、效率等。

### Max_used_connections

Mysql 的 [Max_used_connections](https://dev.mysql.com/doc/refman/8.0/en/server-status-variables.html#statvar_Max_used_connections) 是自服务器启动以来同时使用的最大连接数。

```mysql
mysql> show global status like 'Max_used_connections';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| Max_used_connections | 101   |
+----------------------+-------+
1 row in set (0.00 sec)
```

通过查看这个值，我们就能知道是否需要调整 `max_connections`。

对于 Mysql 最大连接数值的设置范围比较理想的是：服务器响应的最大连接数值占服务器上限连接数值的比例值在10%以上，如果在10%以下，说明最大连接上限值设置过高，反之如果服务器响应的最大连接数值与上限连接数值很接近，则说明最大连接上限值设置过低。

## 解决方案
如果业务量并不大，没有高并发等场景，大部分情况下都是第一种原因，这也是最简单的解决方案，直接修改最大连接数量。\
如果是因为访问量过高，这个时候就需要考虑增加从服务器或分散读压力了（不在本文谈论范围）。

通常修改最大连接数量有两种方式：

1. 直接通过命令行进行修改：
```mysql
mysql> set global max_connections = 1000;
```

使用这种方式需要注意的是，更新之后的最大连接数量，只在 Mysql 当前服务进程有效，也就是说一旦重启 Mysql，又会恢复到初始状态。因为 Mysql 启动后的初始化工作是从其配置文件中读取数据的，而这种方式没有对其配置文件做更改。

2. 修改配置文件：

找到 `my.ini` 或 `my.cnf` 配置文件
```mysql
max_connections=1000
```

重启 Mysql 即可。

##  参考链接
* [Mysql Too many connections](https://dev.mysql.com/doc/refman/8.0/en/too-many-connections.html)
* [mysql 最大连接数概念、作用及修改](https://www.yisu.com/zixun/38410.html)