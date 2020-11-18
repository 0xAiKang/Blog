---
title: Mysql 如何选择 Float、Double、Decimal
date: 2020-11-06 22:25:45
tags: ["Mysql"]
categories: ["Mysql"]
---

我们知道在Mysql 中存储小数有三种数据类型可做选择，究竟该选择哪一种数据格式，其实并没有统一的答案，得根据实际场景去分析，哪一种更合适。

<!-- more -->

## 场景重现

先来看这样一个例子，假设目前有一张表用来存储用户的积分

```
CREATE TABLE `table1` (
  `integral` float(10,2) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

然后向这张表中插入一条数据：

```
mysql> INSERT INTO `table1` (`integral`) VALUES (131072.32);
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM `table1`;
+-----------+
| integral  |
+-----------+
| 131072.31 |
+-----------+
1 row in set (0.00 sec)
```
通过查询数据表可以看到该条记录并不是`131072.32` 而是`131072.31`，为什么会这样？这个问题间接暴露出了其他什么问题？
1. 丢失数据是否是正常现象？
2. 为什么会少0.01，有没有可能少0.02，或者少1，少10甚至少100？
3. 怎么样才能让我们的数据尽量准确？

## 精度是如何丢失的

数值类型存储需求
|列类型|存储需求|分配内存空间|
|-|-|-|
|FLOAT(p)|如果0 <= p <= 24为4个字节, 如果25 <= p <= 53为8个字节|32,64|
|FLOAT|4个字节|32|
|DOUBLE [PRECISION], item REAL|8个字节|64|
|DECIMAL(M,D), NUMERIC(M,D)|变长||

通过查阅[官方文档](https://www.mysqlzh.com/doc/106/276.html)，可以看到
在计算机的世界中，浮点数进行存储时，必须要先转换为二进制，通俗一点讲也就是浮点数的精度实际上是由二进制的精度来决定的。

我们知道对于float类型的数据，只分配了32位的存储空间，对于double类型值分配了64位，但是并不是所有的实数都能转成32位或者64位的二进制形式，**如果超过了，就会出现截断，这就是误差的来源**。

比如将上面例子中的 `131072.32` 转成二进制后的数据为：

```
100000000000000000.0101000111101011100001010001111010111000010100011111…
```

这是一个无穷数，对于float 类型，只能截取前32位进行存储，对于double只能截取前64位进行存储。

* 对于 float 而言，最终存储的值是：`01001000000000000000000000010100`
* 对于 double 而言，最终存储的值是：`0100000100000000000000000000001010001111010111000010100011110101`

所以我们暂时可以得出一个结论：

## 认识Float、Decimal

Float 和 Decimal 这类数据类型都可以通过两位参数来控制其精度。

其存储格式是：
```
FLOAT/DECIMAIL [(M,D)] [UNSIGNED] [ZEROFILL]
```

### 常见误区
1. 精度总能精确到D 位。

存储空间大小决定存储精度，和D值无关，Float 的存储空间只有32 位，当需要存储的二进制大于32 位时，就会截断（四舍五入）。
```
mysql> create table table2 (integral float(15,2));
Query OK, 0 rows affected (0.02 sec)

mysql> insert into table2 values (123456789.39);
Query OK, 1 row affected (0.00 sec)

mysql> select * from table2;
+--------------+
| integral     |
+--------------+
| 123456792.00 |
+--------------+
1 row in set (0.00 sec)
```

2. 数据存储只能存储到D 位

浮点型数据最终都要被转成二进制进行存储。并且对于float 而言，存储类型只能是32位0和1的组合。
```
mysql> select * from table1;
+-----------+
| integral  |
+-----------+
| 131072.31 |
+-----------+
1 row in set (0.00 sec)

mysql> alter table table1 modify integral float(10,4);
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from table1;
+-------------+
| integral    |
+-------------+
| 131072.3125 |
+-------------+
1 row in set (0.00 sec)
```
`DECIMAL(M,D)`中，D 值的是小数部分的位数。可以看到，当修改了D 的值，这个时候可以看到MySQL 真正存储的数值也发生了变化。

总结：
1. 若插入的值未指定小数部分或者小数部分不足D 位则会自动补到D 位小数。
2. 若插入的值小数部分超过了D 为则会发生截断，截取前D 位小数(四舍五入截取)。
3. M 值指是整数部分加小数部分的总长度，也即插入的数字整数部分不能超过M-D 位，否则不能成功插入，会报超出范围的错误。

### 如何选择Float、Double、Decimal

### 参考链接
* [MySQL如何选择float, double, decimal](http://blog.leanote.com/post/weibo-007/mysql_float_double_decimal)