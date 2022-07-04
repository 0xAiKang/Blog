---
title: order by是怎么工作的
date: 2022-07-04 20:59:02
tags: ["Mysql"]
categories: ["Mysql"]
---

本文是基于 [极客时间——MySQL 实战 45 讲](https://time.geekbang.org/column/intro/100020801) 整理的学习笔记，仅供学习参考，请勿用于商业用途，如若侵权，请联系并删除。

课程重点：
* 了解MySQL 里面 order by 语句的几种算法流程
* 了解几者之间的区别

<!-- more -->

## 全字段排序
从前面的内容中了解到了，为了避免全表扫猫，会在相关的字段上增加索引。

假设有一张这样的表：
```mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```

对于下面这个语句，索引在 city 字段上面，我们用 explain 命令来看看这个语句的执行情况。

```mysql
select city,name,age from t where city='杭州' order by name limit 1000;
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220704205956.png)
Extra 这个字段中的“Using filesort”表示的就是需要排序，MySQL 会给每个线程分配一块内存用于排序，称为 sort_buffer。

为了说明这个 SQL 查询语句的执行过程，先来看一下 city 这个索引的示意图。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220704210012.png)
从图中可以看到，满足 city='杭州’条件的行，是从 ID_X 到 ID_(X+N) 的这些记录。

通常情况下，这个语句执行流程如下所示 ：
1. 初始化 sort_buffer，确定放入 name、city、age 这三个字段
2. 从索引 city 查找第一个满足 city="杭州" 这个条件的主键 id，也就是图中的 ID_X
3. 到主键 ID 索引取出整行，取 name、city、age 三个字段的值，存入 sort_buffer 中
4. 从索引 city 查找下一个满足条件的主键 ID
5. 重复步骤三、步骤四，直到 city 的值不满足查询条件为止，对应的主键 id 也就是图中的 ID_Y
6. 对 sort_buffer 中的数据按照字段 name 做快速排序
7. 按照排序结果取前 1000 行返回给客户端

暂且把这个排序过程，称为全字段排序，执行流程的示意图如下所示，下一篇文章中我们还会用到这个排序。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220704210038.png)

图中“按 name 排序”这个动作，可能在内存中完成，也可能需要使用外部排序，这取决于排序所需的内存和参数 sort_buffer_size。

sort_buffer_size，就是 MySQL 为排序开辟的内存（sort_buffer）的大小。如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序。

## rowid 排序
在上面这个算法过程里面，只对原表的数据读了一遍，剩下的操作都是在 sort_buffer 和临时文件中执行的。但这个算法有一个问题，就是如果查询要返回的字段很多的话，那么 sort_buffer 里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差。

所以如果单行很大，这个方法效率不够好，

新的算法放入 sort_buffer 的字段，只有要排序的列（即 name 字段）和主键 id。

但这时，排序的结果就因为少了 city 和 age 字段的值，不能直接返回了，整个执行流程就变成如下所示的样子：
1. 初始化 sort_buffer，确定放入两个字段，即 name 和 id
2. 从索引 city 找到第一个满足 city='杭州’条件的主键 id，也就是图中的 ID_X
3. 到主键 id 索引取出整行，取 name、id 这两个字段，存入 sort_buffer 中
4. 从索引 city 取下一个记录的主键 id
5. 重复步骤 3、4 直到不满足 city='杭州’条件为止，也就是图中的 ID_Y
6. 对 sort_buffer 中的数据按照字段 name 进行排序
7. 遍历排序结果，取前 1000 行，并按照 id 的值回到原表中取出 city、name 和 age 三个字段返回给客户端

这个执行流程的示意图如下，暂且把这个排序过程，称为 rowid 排序

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220704210130.png)
对比上面的全字段排序流程图可以发现，rowid 排序多访问了一次表 t 的主键索引，也就是步骤 7。

## 全字段排序 VS rowid 排序
两种排序方式的区别：
* 如果 MySQL 实在是担心排序内存太小，会影响排序效率，才会采用 rowid 排序算法，这样排序过程中一次可以排序更多行，但是需要再回到原表去取数据。
* 如果 MySQL 认为内存足够大，会优先选择全字段排序，把需要的字段都放到 sort_buffer 中，这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据。

这也就体现了 MySQL 的一个设计思想：**如果内存够，就要多利用内存，尽量减少磁盘访问**。

对于 InnoDB 表来说，rowid 排序会要求回表多造成磁盘读，因此不会被优先选择。

其实并不是所有的 order by 语句，都需要排序操作的。

从上面分析的执行过程，我们可以看到，MySQL 之所以需要生成临时表，并且在临时表上做排序操作，**其原因是原来的数据都是无序的**。

也就是说如果能够保证从索引上取出来的行，天然就是按某个顺序（递增或者递减）排列的话，那就可以不用再排序了。

## 总结
* Mysql 的设计思想：如果内存够，就要多利用内存，尽量减少磁盘访问
* 在 Mysql 中做排序，是一个成本比较高的操作，但也不是所有 order by 都需要排序，只有数据是无序的时候，才会生成临时表，并在临时表上面进行排序
* InnoDB 有两种排序方式：全字段排序和rowid 排序，rowid 排序会要求回表多造成磁盘读，通常不会被优先选择