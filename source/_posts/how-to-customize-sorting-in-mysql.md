---
title: Mysql 如何自定义排序
date: 2021-07-26 08:27:30
tags: ["Mysql"]
categories: ["Mysql"]
---

在Mysql 中，想要对结果进行排序，通常会使用`Order By` 子句，使用时，应注意以下几点：
* 在Order By 子句中，使用 ASC 或 DESC 关键字来设置查询结果是按升序或降序排列; 默认按升序（ASC）排列
* Order By 子句可以指定多个排序键
* 存在多个排序键时，优先使用左侧的键，如果该列存在相同值的话，则按右侧的键进行排列
* 如果排序键的值包含NULL，则会在开头或者结尾进行汇总（按最小值对待）

<!-- more -->

> 存在多个排序键时需要注意，第一列必须有相同的值，才会对第二个列进行排序; 如果第一个列的所有值都是唯一的，那么Mysql 将不再对第二个列进行排序。

这篇笔记的重点是记录如何在Mysql中 使用自定义排序。

为什么会有这样的需求呢？

在回答这个问题之前，先来看这样一个场景。

假设现在有一张审核记录表：

| 字段名称   | 字段类型   | 字段默认值    | 是否允许为空 | 索引     | 示例值              | 字段描述                              |
| ---------- | ---------- | ------------- | ------------ | -------- | ------------------- | ------------------------------------- |
| id         | Bigint(16) | Unsigned 自增 | 否           | 主键     | 1                   | 主键 ID                               |
| uid        | Bigint(16) | 0             | 否           | 普通索引 | 1                   | 用户 ID                               |
| status     | Tinyint(1) | 0             | 否           | -        | 0                   | 审核状态（1. 等待审核 2. 审核通过 3. 审核拒绝） |
| created_at | Timestamp  |               | 否           | -        | 2021-06-03 21:54:53 | 创建时间                              |
| updated_at | Timestamp  |               | 是           | -        | 2021-06-03 21:54:57 | 更新时间                              |

原本是按照这样的顺序进行排序的：`待审核=> 审核通过=> 审核拒绝`

对应SQL 应该是：
```mysql
SELECT * FROM table
ORDER BY status
```

假如有一天需求发生了变化，需要优先将审核通过的排在前面，其次是等待审核，最后才是审核拒绝。

此时，你肯定不想将已经写好的代码再“重构”一次，手动将`待审核` 与 `审核通过` 的顺序进行调换。一来是，可能会将现有完整的功能，改出问题，二来是，如果后面排序需求再次发生变化，就得再次面临相同的问题。

> 如果Mysql 能自定义排序，那该多好啊。

不禁会这样去想。

可以使用Mysql 的字符串函数——[FIELD](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_field)，在不改变现有逻辑的基础上，仅仅只改变排序顺序。

对应SQL 如下：
```mysql
SELECT * FROM table
ORDER BY FIELD(status, 2, 1, 3)
```
