---
title: MySQL Integer类型与INT(11)详解
date: 2020-10-23 18:39:10
tags: ["Mysql"]
categories: ["Mysql"]
---

MySQL支持的整数类型有TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT。

<!-- more -->

每种整数类型所需的存储空间和范围如下：
|类型|字节|最小值(有符号)|最大值(有符号)|最小值(无符号)|最大值(无符号)|
|-|-|-|-|-|-|
|TINYINT|1|-128|127|0|255|
|SMALLINT|2|-32768|32767|0|65535|
|MEDIUMINT|3|-8388608|8388607|0|16777215|
|INT|4|-2147483648|2147483647|0|4294967295|
|BIGINT|8|-9223372036854775808|(9223372036854775807|0|18446744073709551615|

## 有无限制的区别
在创建数据表时，通常会看见 `int(11)`和`int`这样的写法，这两者有什么区别，各自又代表什么意思呢？

1. 对应Integer 类型而言，仅表示字段的显示宽度。
2. 对于DECIMAL类型，表示数字的总数。
3. 对于字符字段，这是可以存储的最大字符数，例如VARCHAR（20）可以存储20个字符。

**显示宽度并不影响可以存储在该列中的最大值。**`int(3)`和`int(11)` 所能存储的最大范围是一样的。

将某个字段设置成`INT(20)`并不意味着将能够存储20位数字，这个字段最终能存储的最大范围还是 INT 的范围。

### 示例
创建一张临时表：
```
CREATE TABLE tmp_table_a (
    id INT(3) NOT NULL AUTO_INCREMENT,
    name varchar(16) DEFAULT '' NOT NULL, 
    PRIMARY KEY (`id`)
);
```

查看表结构：
```
mysql> desc tmp_table_a;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(3)      | NO   | PRI | NULL    | auto_increment |
| name  | varchar(16) | NO   |     |         |                |
+-------+-------------+------+-----+---------+----------------+
```

插入超过"长度"的数字：
```
INSERT INTO tmp_table_a(id, name) VALUES(123456, "boo");
```

查看结果，发现数字并没有插入失败：
```
mysql> select * from tmp_table_a;
+--------+------+
| id     | name |
+--------+------+
| 123456 | boo  |
+--------+------+
1 row in set (0.00 sec)
```

## 有无符号的区别

那么问题来了，既然加不加数字并没有什么区别，那为什么还多此一举呢？

这是因为“正常”情况下确实没有什么区别，只有当**字段设置为UNSIGNED ZEROFILL 属性时**，为INT 增加数字才会有意义。

表示如果要存储的数字少于N 个字符，则这些数字将在左侧补零。

### 示例
创建一张 UNSIGNED ZEROFILL 的数据表：
```
CREATE TABLE tmp_table_b (
    id INT(3) UNSIGNED ZEROFILL NOT NULL AUTO_INCREMENT,
    name varchar(16) DEFAULT '' NOT NULL, 
    PRIMARY KEY (`id`)
);
```

查看表结构：
```
mysql> desc tmp_table_b;
+-------+--------------------------+------+-----+---------+----------------+
| Field | Type                     | Null | Key | Default | Extra          |
+-------+--------------------------+------+-----+---------+----------------+
| id    | int(3) unsigned zerofill | NO   | PRI | NULL    | auto_increment |
| name  | varchar(16)              | NO   |     |         |                |
+-------+--------------------------+------+-----+---------+----------------+
```

插入记录：
```
INSERT INTO tmp_table_b(id, name) VALUES(1, "boo");
```

查看记录：
```
mysql> select * from tmp_table_b;
+-----+------+
| id  | name |
+-----+------+
| 001 | boo  |
+-----+------+
```

### 总结
1. 对于Integer 类型而言，“数字”并不会限制其能存储的最大范围。
2. 有无符号，不仅会限制其能存储的最大范围，还可以配置“数字”自动补零。

### 参考链接
* [MySQL Integer类型与INT(11)](https://www.cnblogs.com/polk6/p/11595107.html)