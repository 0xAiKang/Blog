---
title: 13 为什么表数据删掉一半，表文件大小不变
date: 2022-06-23 22:29:12
tags: ["Mysql"]
categories: ["Mysql"]
---

本文是基于 [极客时间——MySQL 实战 45 讲](https://time.geekbang.org/column/intro/100020801) 整理的学习笔记，仅供学习参考，请勿用于商业用途，如若侵权，请联系并删除。

课程重点：
* 了解 Mysql 的innodb_file_per_table 参数
* 了解 Mysql 数据删除流程
* 区分 Online 和 Inplace

<!-- more -->

## 参数 innodb_file_per_table

表数据既可以存在共享表空间里，也可以是单独的文件。这个行为是由参数 innodb_file_per_table 控制的：
1. 这个参数设置为 OFF 表示的是，表的数据放在系统共享表空间，也就是跟数据字典放在一起
2. 这个参数设置为 ON 表示的是，每个 InnoDB 表数据存储在一个以 .ibd 为后缀的文件中

从 MySQL 5.6.6 版本开始，它的默认值就是 ON 了。

建议不论使用 MySQL 的哪个版本，都将这个值设置为 ON。

因为，一个表单独存储为一个文件更容易管理，而且在你不需要这个表的时候，通过 `drop table` 命令，系统就会直接删除这个文件。而如果是放在共享表空间中，即使表删掉了，空间也是不会回收的。

## 数据删除流程
B + 树索引示意图：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220623222553.png)

假设，我们要删掉 R4 这个记录，InnoDB 引擎只会把 R4 这个记录标记为删除。如果之后要再插入一个 ID 在 300 和 600 之间的记录时，可能会复用这个位置。但是，磁盘文件的大小并不会缩小。

在 InnoBD 中，数据是按页进行存储的，如果我们删掉了一个数据页上的所有记录，会怎么样？

答案是，整个数据页就可以被复用了。

但是，**数据页的复用跟记录的复用是不同的**。

**记录的复用，只限于符合范围条件的数据**。比如上面的这个例子，R4 这条记录被删除后，如果插入一个 ID 是 400 的行，可以直接复用这个空间。但如果插入的是一个 ID 是 800 的行，就不能复用这个位置了。

**数据页的复用**，当整个页从 B+ 树里面摘掉以后，可以复用到任何位置。以上图为例，如果将数据页 page A 上的所有记录删除以后，page A 会被标记为可复用。这时候如果要插入一条 ID=50 的记录需要使用新页的时候，page A 是可以被复用的。

如果相邻的两个数据页利用率都很小，系统就会把这两个页上的数据合到其中一个页上，另外一个数据页就被标记为可复用。

进一步地，如果我们用 delete 命令把整个表的数据删除呢？结果就是，所有的数据页都会被标记为可复用。但是磁盘上，文件不会变小。

你现在知道了，delete 命令其实只是把记录的位置，或者数据页标记为了“可复用”，但磁盘文件的大小是不会变的。也就是说，通过 delete 命令是不能回收表空间的。这些可以复用，而没有被使用的空间，看起来就像是“空洞”。

实际上，**不止是删除数据会造成空洞，插入数据也会**。

如果数据是按照索引递增顺序插入的，那么索引是紧凑的。但如果数据是随机插入的，就可能造成索引的数据页分裂。

假设图 1 中 page A 已经满了，这时我要再插入一行数据，会怎样呢？

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220623222611.png)

可以看到，由于 page A 满了，再插入一个 ID 是 550 的数据时，就不得不再申请一个新的页面 page B 来保存数据了。页分裂完成后，page A 的末尾就留下了空洞（注意：实际上，可能不止 1 个记录的位置是空洞）。

另外，更新索引上的值，可以理解为删除一个旧的值，再插入一个新值。不难理解，这也是会造成空洞的。

也就是说，经过大量增删改的表，都是可能是存在空洞的。所以，如果能够把这些空洞去掉，就能达到收缩表空间的目的。

而重建表，就可以达到这样的目的。

## 重建表
可以新建一个与表 A 结构相同的表 B，然后按照主键 ID 递增的顺序，把数据一行一行地从表 A 里读出来再插入到表 B 中。

由于表 B 是新建的表，所以表 A 主键索引上的空洞，在表 B 中就都不存在了。显然地，表 B 的主键索引更紧凑，数据页的利用率也更高。如果我们把表 B 作为临时表，数据从表 A 导入表 B 的操作完成后，用表 B 替换 A，从效果上看，就起到了收缩表 A 空间的作用。

可以使用 alter table A engine=InnoDB 命令来重建表。在 MySQL 5.5 版本之前，这个命令的执行流程跟我们前面描述的差不多，区别只是这个临时表 B 不需要你自己创建，MySQL 会自动完成转存数据、交换表名、删除旧表的操作。

不过需要注意的是，在整个 DDL 过程中，表 A 中不能有更新。也就是说，这个 DDL 不是 Online 的。

## Online 和 Inplace
另一个跟 Online 有关， 比较容易混淆的概念是 Inplace。

在上面的重建表的过程中，重建出来的数据会放在“tmp_file” 临时文件中，这个临时文件是 InnoDB 在内部创建出来的。整个 DDL 过程都在 InnoDB 内部完成。

对于 server 层来说，**没有把数据挪动到临时表，是一个“原地”操作，这就是“Inplace”名称的来源**。

所以，我现在问你，如果你有一个 1TB 的表，现在磁盘间是 1.2TB，能不能做一个 Inplace 的 DDL 呢？

答案是不能。因为，tmp_file 也是要占用临时空间的。

Online 和 Inplace 这两个逻辑之间的关系是什么？
* DDL 过程如果是 Online 的，就一定是 inplace 的
* 反过来未必，也就是说 inplace 的 DDL，有可能不是 Online 的。截止到 MySQL 8.0，添加全文索引（FULLTEXT index）和空间索引 (SPATIAL index) 就属于这种情况

## 总结
* 只是 delete 掉表里面不用的数据的话，表文件的大小是不会变的
* drop table 命令可以回收表空间，但是会把表结构定义和数据全部删掉
* 如果希望在保留表结构以及数据的情况下，还能回收表空间，那么可以考虑重建表
* 使用 alter table 重建表，有两种方式：
  * `alter table t engine=innodb,ALGORITHM=copy;`：这个 DDL 不是 Online 的
  * `alter table t engine=innodb,ALGORITHM=inplace;`：这个 DDL 是 Online 的
* Mysql 5.6 之后，除了增加加全文索引不是 online，其他 alter 操作(增删字段、增删索引等)都是支持 online ddl 