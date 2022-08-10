---
title: 17 如何正确地显示随机消息
date: 2022-08-10 10:26:51
tags: ["Mysql"]
categories: ["Mysql"]
---

本文是基于 [极客时间——MySQL 实战 45 讲](https://time.geekbang.org/column/intro/100020801) 整理的学习笔记，仅供学习参考，请勿用于商业用途，如若侵权，请联系并删除。

课程重点：
* 了解 `order by rand` 背后的执行流程
* 了解内存临时表和磁盘临时表

<!-- more -->

## 内存临时表
Mysql 中的 `rand()` 函数通常用来做随机排序。

```mysql
select word from words order by rand() limit 3;
```

这个语句的意思很直白，随机排序取前 3 个。虽然这个 SQL 语句写法很简单，但执行流程却有点复杂的。

先用 explain 命令来看看这个语句的执行情况：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220810101905.png)
Extra 字段显示 Using temporary，表示的是需要使用临时表；Using filesort，表示的是需要执行排序操作。

因此这个 Extra 的意思就是，需要临时表，并且需要在临时表上排序。

这条语句的执行流程是这样的：
1. 创建一个临时表。这个临时表使用的是 memory 引擎，表里有两个字段，第一个字段是 double 类型，为了后面描述方便，记为字段 R，第二个字段是 varchar(64) 类型，记为字段 W。并且，这个表没有建索引。
2. 从 words 表中，按主键顺序取出所有的 word 值。对于每一个 word 值，调用 rand() 函数生成一个大于 0 小于 1 的随机小数，并把这个随机小数和 word 分别存入临时表的 R 和 W 字段中，到此，扫描行数是 10000。
3. 现在临时表有 10000 行数据了，接下来你要在这个没有索引的内存临时表上，按照字段 R 排序。
4. 初始化 sort_buffer。sort_buffer 中有两个字段，一个是 double 类型，另一个是整型。
5. 从内存临时表中一行一行地取出 R 值和位置信息（我后面会和你解释这里为什么是“位置信息”），分别存入 sort_buffer 中的两个字段里。这个过程要对内存临时表做全表扫描，此时扫描行数增加 10000，变成了 20000。
6. 在 sort_buffer 中根据 R 的值进行排序。注意，这个过程没有涉及到表操作，所以不会增加扫描行数。
7. 排序完成后，取出前三个结果的位置信息，依次到内存临时表中取出 word 值，返回给客户端。这个过程中，访问了表的三行数据，总扫描行数变成了 20003。

查看慢查询日志（slow log，将 long_query_time 的时间设置为 0，这样所有的查询都会被记录到）来验证一下分析得到的扫描行数是否正确：
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220810102023.png)

Rows_examined：20003 就表示这个语句执行过程中扫描了 20003 行，也就验证了我们分析得出的结论。

随机排序完整流程图：
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220810101925.png)
图中的 pos 就是位置信息，你可能会觉得奇怪，这里的“位置信息”是个什么概念？在上一篇文章中，我们对 InnoDB 表排序的时候，明明用的还是 ID 字段。

这时候，我们就要回到一个基本概念：MySQL 的表是用什么方法来定位“一行数据”的。

如果创建的表没有主键，或者把一个表的主键删掉了，那么 InnoDB 会自己生成一个长度为 6 字节的 rowid 来作为主键。

这也就是排序模式里面，rowid 名字的来历。实际上它表示的是：每个引擎用来唯一标识数据行的信息。
1. 对于有主键的 InnoDB 表来说，这个 rowid 就是主键 ID；
2. 对于没有主键的 InnoDB 表来说，这个 rowid 就是由系统生成的；
3. MEMORY 引擎不是索引组织表。在这个例子里面，你可以认为它就是一个数组。因此，这个 rowid 其实就是数组的下标

**order by rand() 使用了内存临时表，内存临时表排序的时候使用了 rowid 排序方法**。

## 磁盘临时表
那么，是不是所有的临时表都是内存表呢？

是的。tmp_table_size 这个配置限制了内存临时表的大小，默认值是 16M。如果临时表大小超过了 tmp_table_size，那么内存临时表就会转成磁盘临时表。

磁盘临时表使用的引擎默认是 InnoDB，是由参数 internal_tmp_disk_storage_engine 控制的。

## 随机排序方法
我们先把问题简化一下，如果只随机选择 1 个 word 值，可以怎么做呢？思路上是这样的：

1. 取得这个表的主键 id 的最大值 M 和最小值 N
2. 用随机函数生成一个最大值到最小值之间的数 X = (M-N)*rand() + N
3. 取不小于 X 的第一个 ID 的行

这个算法暂时称作随机算法 1。下面是执行语句的序列：
```mysql
select max(id),min(id) into @M,@N from words ;
set @X= floor((@M-@N+1)*rand() + @N);
select * from words where id >= @X limit 1;
```

这个方法效率很高，因为取 max(id) 和 min(id) 都是不需要扫描索引的，而第三步的 select 也可以用索引快速定位，可以认为就只扫描了 3 行。但实际上，这个算法本身并不严格满足题目的随机要求，因为 ID 中间可能有空洞，因此选择不同行的概率不一样，不是真正的随机。

比如你有 4 个 id，分别是 1、2、4、5，如果按照上面的方法，那么取到 id=4 的这一行的概率是取得其他行概率的两倍。

如果这四行的 id 分别是 1、2、40000、40001 呢？这个算法基本就能当 bug 来看待了。

所以，为了得到严格随机的结果，可以用下面这个流程:
1. 取得整个表的行数，并记为 C
2. 取得 Y = floor(C * rand())。 floor 函数在这里的作用，就是取整数部分
3. 再用 limit Y,1 取得一行

这个算法暂时称作随机算法 2。下面是执行语句的序列：
```mysql
select count(*) into @C from words;
set @Y = floor(@C * rand());
set @sql = concat("select * from words limit ", @Y, ",1");

prepare stmt from @sql;
execute stmt;
DEALLOCATE prepare stmt;
```
由于 limit 后面的参数不能直接跟变量，所以上述执行序列中，使用了 prepare+execute 的方法。你也可以把拼接 SQL 语句的方法写在应用程序中，会更简单些。

现在再看看，如果按照随机算法 2 的思路，要随机取 3 个 word 值呢？你可以这么做：
1. 取得整个表的行数，记为 C；
2. 根据相同的随机方法得到 Y1、Y2、Y3；
3. 再执行三个 limit Y, 1 语句得到三行数据。

把这个算法，称作随机算法 3。下面是执行语句的序列：
```mysql
select count(*) into @C from t;
set @Y1 = floor(@C * rand());
set @Y2 = floor(@C * rand());
set @Y3 = floor(@C * rand());

# 在应用代码里面取 Y1、Y2、Y3 值，拼出 SQL 后执行
select * from t limit @Y1，1；
select * from t limit @Y2，1；
select * from t limit @Y3，1;
```

## 总结
* 遇到随机排序需求时，应尽量避免使用 `order by rand()`，因为这个语句需要 Using temporary 和 Using filesort，查询的执行代价往往是比较大的。
* 正确的做法应该是，使用后面的几个算法方案，通过拼接 SQL 语句，获取预期的结果集。