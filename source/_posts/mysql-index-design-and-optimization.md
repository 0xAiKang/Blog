---
title: Mysql 索引设计与优化
date: 2021-01-10 21:12:04
tags: ["Mysql"]
categories: ["Mysql"]
---

什么是索引？

<!-- more -->

> 数据库索引是一种数据结构，它以额外的写入和存储空间为代价来提高数据库表上数据检索操作的速度。通俗来说，索引类似于书的目录，根据其中记录的页码可以快速找到所需的内容。——维基百科

常见索引有哪些？
* 普通索引：最基本的索引，没有任何限制
* 唯一索引：与”普通索引“类似，不同的就是：索引列的值必须是唯一，但允许有空值
* 主键索引：它是一种特殊的索引，不允许有空值
* 全文索引：仅可用于 MyISAM 表，针对较大的数据，生成全文索引很耗时占空间
* 组合索引：为了提高多条件查询效率，可建立组合索引，遵循”最左前缀匹配原则“

这里以相对复杂的组合为例，介绍如何优化。

## 最左前缀匹配原则
首先我们要知道什么是最左前缀匹配原则。

最左前缀匹配原则是指在使用 B+Tree 联合索引进行数据检索时，MySQL 优化器会读取谓词（过滤条件）**并按照联合索引字段创建顺序一直向右匹配直到遇到范围查询或非等值查询后停止匹配**，此字段之后的索引列不会被使用，这时计算 `key_len` 可以分析出联合索引实际使用了哪些索引列。

### 如何计算 key_len
通过 `key_len` 计算也帮助我们了解索引的最左前缀匹配原则。

`key_len` 表示得到结果集所使用的选择索引的长度[字节数]，不包括 `order by`，也就是说如果 `order by` 也使用了索引则 `key_len` 不计算在内。

在计算 `key_len` 之前，先来温习一下基本数据类型（以UTF8 编码为例）：
|类型|所占空间|不允许为NULL额外占用|
|-|-|-|
|char|一个字符三个字节|一个字节|
|varchar|一个字符三个字节|一个字节|
|int|四个字节|一个字节|
|tinyint|一个字节|一个字节|

测试数据表如下：
```
CREATE TABLE `test_table` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a` int(11) DEFAULT NOT NULL,
  `b` int(11) DEFAULT NOT NULL,
  `c` int(11) DEFAULT NOT NULL,
  PRIMARY KEY (`id`),
  KEY `test_table_a_b_c_index` (`a`,`b`,`c`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

命中索引：
```
mysql> explain select * from test_table where a = 1 and b = 2 and c = 3;
+----+-------------+------------+------------+------+------------------------+------------------------+---------+-------------------+------+----------+-------------+
| id | select_type | table      | partitions | type | possible_keys          | key                    | key_len | ref               | rows | filtered | Extra       |
+----+-------------+------------+------------+------+------------------------+------------------------+---------+-------------------+------+----------+-------------+
|  1 | SIMPLE      | test_table | NULL       | ref  | test_table_a_b_c_index | test_table_a_b_c_index | 12      | const,const,const |    1 |   100.00 | Using index |
+----+-------------+------------+------------+------+------------------------+------------------------+---------+-------------------+------+----------+-------------+
```
可以看到 `key_len = 12`，这是如何计算的呢？
因为字符集是 UTF8，一个字段占用四个字节，三个字段就是 4 * 3 = 12 字节。

是否允许为 NULL，如果允许为 NULL，则需要用额外的字节来标记该字段，不同的数据类型所需的字节大小不同。
```
mysql> ALTER TABLE `test_table` CHANGE `a` `a` INT(11)  NULL;
mysql> ALTER TABLE `test_table` CHANGE `c` `c` INT(11)  NULL;
mysql> ALTER TABLE `test_table` CHANGE `b` `b` INT(11)  NULL;
mysql> explain select * from test_table where a = 1 and b = 2 and c = 3;
+----+-------------+------------+------------+------+------------------------+------------------------+---------+-------------------+------+----------+-------------+
| id | select_type | table      | partitions | type | possible_keys          | key                    | key_len | ref               | rows | filtered | Extra       |
+----+-------------+------------+------------+------+------------------------+------------------------+---------+-------------------+------+----------+-------------+
|  1 | SIMPLE      | test_table | NULL       | ref  | test_table_a_b_c_index | test_table_a_b_c_index | 15      | const,const,const |    1 |   100.00 | Using index |
+----+-------------+------------+------------+------+------------------------+------------------------+---------+-------------------+------+----------+-------------+
```
可以看到，当字段允许为空时，这时的`key_len` 变成了15 = 4 * 3 + 1 * 3（INT 类型为空时，额外占用一个字节）。

## 索引优化
有了这些基础知识之后，再来根据实际的SQL 判断索性性能好坏。

还是以上面那张数据表为例，为 a、b、c 三个字段创建联合索引。
|SQL 语句|是否索引|
|-|-|
|explain select * from test_table where a = 1 and b = 2 and c = 3;|Extra:Using index key_len: 15|
|explain select * from test_table where a = 1 and b = 2 and c = 3 order by c;|Extra:Using index key_len: 15|
|explain select * from test_table where b = 2 and c = 3;|Extra:Using where; Using index key_len: 15|
|explain select * from test_table where a = 1 order by c;|Extra:Using where; Using index; Using filesort key_len: 5|
|explain select * from test_table order by a, b, c;|Extra:Using index key_len: 15|
|explain select * from test_table order by a, b, c desc;|Extra:Using index; Using filesort key_len:15|
|explain select * from test_table where a in (1,2) and b in (1,2,3) and c = 1;|Extra:Using where; Using index key_len: 15|

通常在查看执行计划时， Extra 列为 Using index 则表示优化器使用了覆盖索引。

* SQL1 可以使用覆盖索引，性能好
* SQL2 可以使用覆盖索引，同时避免排序，性能好
* SQL3 可以使用覆盖索引，但是需要根据 where 字句进行过滤
* SQL4 可以使用部分索引 a，但无法避免排序，性能差
* SQL5 可以完全使用覆盖索引，同时可以避免排序，性能好
* SQL6 可以使用覆盖索引，但无法避免排序，（这是因为 MySQL InnoDB 创建索引时默认asc升序，索引无法自动倒序排序）
* SQL7 可以使用覆盖索引，但是需要根据 where 子句进行过滤（非定值查询）

## 创建索引规范
* 考虑到索引维护的成本，单张表的索引数量不超过 5 个，单个索引中的字段数不超过 5 个
* 不在低基数列上建⽴索引，例如“性别”。 在低基数列上创建的索引查询相比全表扫描不一定有性能优势，特别是当存在回表成本时。
* 合理创建联合索引，(a,b,c) 相当于 (a) 、(a,b) 、(a,b,c)。 
* 合理使用覆盖索引减少IO，避免排序。

### 参考链接
* [Explain之key_len长度计算](https://www.cnblogs.com/xuanzhi201111/p/4554769.html)