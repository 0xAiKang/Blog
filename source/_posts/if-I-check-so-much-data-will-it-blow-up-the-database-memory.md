---
title: 33 我查这么多数据，会不会把数据库内存打爆
date: 2022-08-25 11:00:58
tags: ["Mysql"]
categories: ["Mysql"]
---

本文是基于 [极客时间——MySQL 实战 45 讲](https://time.geekbang.org/column/intro/100020801) 整理的学习笔记，仅供学习参考，请勿用于商业用途，如若侵权，请联系并删除。

课程重点：
* 了解全部扫描对 Server 层的影响

<!-- more -->

## 全表扫描对 server 层的影响
假设，现在需要对一个 200G 的 InnoDB 表 db1. t，执行一个全表扫描。当然，你要把扫描结果保存在客户端，会使用类似这样的命令：

```mysql
mysql -h$host -P$port -u$user -p$pwd -e "select * from db1.t" > target_file.sql
```

InnoDB 的数据是保存在主键索引上的，所以全表扫描实际上是直接扫描表 t 的主键索引。这条查询语句由于没有其他的判断条件，所以查到的每一行都可以直接放到结果集里面，然后返回给客户端。

那么，这个“结果集”存在哪里呢？

实际上，服务端并不需要保存一个完整的结果集。取数据和发数据的流程是这样的：
1. 获取一行，写到 net_buffer 中。这块内存的大小是由参数 net_buffer_length 定义的，默认是 16k。
2. 重复获取行，直到 net_buffer 写满，调用网络接口发出去。
3. 如果发送成功，就清空 net_buffer，然后继续取下一行，并写入 net_buffer。
3. 如果发送函数返回 EAGAIN 或 WSAEWOULDBLOCK，就表示本地网络栈（socket send buffer）写满了，进入等待。直到网络栈重新可写，再继续发送。

这个过程对应的流程图如下所示。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220816143632.png)

从这个流程中，可以看到：
1. 一个查询在发送过程中，占用的 MySQL 内部的内存最大就是 net_buffer_length 这么大，并不会达到 200G
2. socket send buffer 也不可能达到 200G（默认定义 /proc/sys/net/core/wmem_default），如果 socket send buffer 被写满，就会暂停读数据的流程。

也就是说，**MySQL 是“边读边发的”**，这个概念很重要。这就意味着，**如果客户端接收得慢，会导致 MySQL 服务端由于结果发不出去，这个事务的执行时间变长**。

比如下面这个状态，就是我故意让客户端不去读 socket receive buffer 中的内容，然后在服务端 show processlist 看到的结果。

![服务端发送阻塞](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220816143838.png)

如果你看到 State 的值一直处于“Sending to client”，就表示服务器端的网络栈写满了。

在上一篇文章中曾提到，如果客户端使用–quick 参数，会使用 mysql_use_result 方法。这个方法是读一行处理一行。你可以想象一下，假设有一个业务的逻辑比较复杂，每读一行数据以后要处理的逻辑如果很慢，就会导致客户端要过很久才会去取下一行数据，可能就会出现上图所示的这种情况。

因此，对于正常的线上业务来说，如果一个查询的返回结果不会很多的话，我都建议你使用 mysql_store_result 这个接口，直接把查询结果保存到本地内存（Mysql 默认正是 mysql_store_result 这个接口）。

另一方面，如果你在自己负责维护的 MySQL 里看到很多个线程都处于“Sending to client”这个状态，就意味着你要让业务开发同学优化查询结果，并评估这么多的返回结果是否合理。

而如果要快速减少处于这个状态的线程的话，将 net_buffer_length 参数设置为一个更大的值是一个可选方案。

与“Sending to client”长相很类似的一个状态是“Sending data”，这是一个经常被误会的问题。有同学问我说，在自己维护的实例上看到很多查询语句的状态是“Sending data”，但查看网络也没什么问题啊，为什么 Sending data 要这么久？

实际上，一个查询语句的状态变化是这样的（注意：省略了其他无关的状态）：
1. MySQL 查询语句进入执行阶段后，首先把状态设置成“Sending data”；
2. 然后，发送执行结果的列相关的信息（meta data) 给客户端；
3. 再继续执行语句的流程；
4. 执行完成后，把状态设置成空字符串。

也就是说，“Sending data”并不一定是指“正在发送数据”，而可能是处于执行器过程中的任意阶段。比如，你可以构造一个锁等待的场景，就能看到 Sending data 状态。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220816145620.png)
可以看到，session B 明显是在等锁，状态却显示为 Sending data。

MySQL 采用的是边算边发的逻辑，因此对于数据量很大的查询结果来说，不会在 server 端保存完整的结果集。所以，如果客户端读结果不及时，会堵住 MySQL 的查询过程，但是不会把内存打爆。

全表扫描还是比较耗费 IO 资源的，所以业务高峰期还是不能直接在线上主库执行全表扫描的。

## 总结
* 查询的结果是分段发给客户端的，因此扫描全表，查询返回大量的数据，并不会把内存打爆。
* 查询返回的结果不多的话，建议使用 mysql_store_result 这个接口（默认），如果查询返回的结果集很多，那么这个时候就需要改用 mysql_use_result 接口了。
* 将 net_buffer_length 的值设置更大，可以解决多个线程都处于“Sending to client” 的状态。
* 仅当一个线程处于“等待客户端接收结果”的状态，才会显示"Sending to client"；而如果显示成“Sending data”，并不一定是指“正在发送数据”，而可能是处于执行器过程中的任意阶段。