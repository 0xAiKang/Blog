---
title: Mysql 如何从全备中恢复指定表数据
date: 2022-04-09 16:15:26
tags: ["Mysql"]
categories: ["Mysql"]
---

备份数据库是常有的需求，通常是整库备份，整库还原。\
但如果只想还原其中部分数据表，该怎么做呢？

<!-- more -->

## 测试前准备

1. mysqldump 备份指定数据库，如：test_dump

```bash
mysqldump -uroot -p mysql -F -R -E --triggers --databases test_dump | gzip >dbtest_$(date +%F).sql.gz;
```

2. 确认备份文件已经生成

```bash
ll

dbtest_2022-04-09.sql.gz
```

3. 删除需要还原的表

查看当前数据库中所有表：

```mysql
mysql> show tables;
+---------------------+
| Tables_in_test_dump |
+---------------------+
| tb                  |
| tb2                 |
+---------------------+
3 rows in set (0.00 sec)
```

查看需要还原的表的数据：
```mysql
mysql> select * from tb2;
+----+------+------+------+
| id | name | val  | memo |
+----+------+------+------+
|  1 | a    |    2 | a2   |
|  2 | a    |    1 | a1   |
|  3 | a    |    3 | a3   |
|  5 | b    |    3 | b3   |
|  6 | b    |    2 | b2   |
|  7 | b    |    4 | b4   |
|  8 | b    |    5 | b5   |
|  9 | b    |    1 | b1   |
+----+------+------+------+
8 rows in set (0.00 sec)
```

删除目标表：
```mysql
mysql> drop table tb2;
Query OK, 0 rows affected (0.01 sec)
```

## 从备份中恢复
1. 从备份文件中找出需要恢复的表的建表语句：

```bash
gunzip -c dbtest_2022-04-09.sql.gz | sed -e '/./{H;$!d;}' -e 'x;/CREATE TABLE `tb2`/!d;q';

DROP TABLE IF EXISTS `tb222`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `tb222` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(10) DEFAULT NULL,
  `val` int(11) DEFAULT NULL,
  `memo` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;
```

2. 从备份文件中找出需要恢复表的数据

```bash
gunzip -c dbtest_2022-04-09.sql.gz | sed -e '/./{H;$!d;}' -e 'x;/CREATE TABLE `tb2`/!d;q'

INSERT INTO `tb222` VALUES (1,'a',2,'a2'),(2,'a',1,'a1'),(3,'a',3,'a3'),(5,'b',3,'b3'),(6,'b',2,'b2'),(7,'b',4,'b4'),(8,'b',5,'b5'),(9,'b',1,'b1');
```

确认了数据之后无误之后，就开始恢复了。

3. 恢复被删除表的表结构

```bash
gunzip -c dbtest_2022-04-09.sql.gz | sed -e '/./{H;$!d;}' -e 'x;/CREATE TABLE `tb2`/!d;q' | mysql -uroot -p test_dump
```

4. 从备份文件中恢复被删除表的数据

```bash
gunzip -c dbtest_2022-04-09.sql.gz | grep --ignore-case  'insert into `tb2`'| mysql -uroot -p test_dump
```

查看目标表，数据已经恢复：

```mysql
mysql> select * from tb2;
+----+------+------+------+
| id | name | val  | memo |
+----+------+------+------+
|  1 | a    |    2 | a2   |
|  2 | a    |    1 | a1   |
|  3 | a    |    3 | a3   |
|  5 | b    |    3 | b3   |
|  6 | b    |    2 | b2   |
|  7 | b    |    4 | b4   |
|  8 | b    |    5 | b5   |
|  9 | b    |    1 | b1   |
+----+------+------+------+
8 rows in set (0.00 sec)
```

> 注意：实际使用时以上命令中的部分文件名或表名需要替换成你自己的。