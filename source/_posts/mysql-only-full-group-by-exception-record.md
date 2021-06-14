---
title: Mysql only_full_group_by 异常记录
date: 2020-07-31 20:17:14
tags: ["Mysql"]
categories: ["Mysql"]
---

最近很频繁的遇到一个Mysql 异常，错误信息如下：

> Expression #5 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'cis.q1.query_date' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

通过错误信息可以看到，是因为 `sql_mode` 引起的。

查看Mysql 当前所使用的 `sql_mode`：
```mysql
select @@sql_mode

+-------------------------+
|       @@sql_mode        |
+-------------------------+
|   ONLY_FULL_GROUP_BY    |
+-------------------------+
```

## sql_mode 配置解析
### ONLY_FULL_GROUP_BY
对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中。简而言之，就是SELECT后面接的列必须被GROUP BY后面接的列所包含。如：

```mysql
❎
select a,b from table group by a,b,c; 

✅
select a,b,c from table group by a,b; 
```

这个配置会使得GROUP BY语句环境变得十分狭窄，所以一般都不加这个配置

### NO_AUTO_VALUE_ON_ZERO
该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。（不信的可以试试，默认的sql_mode你在自增主键列设置为0，该字段会自动变为最新的自增值，效果和null一样），如果用户希望插入的值为0（不改变），该列又是自增长的，那么这个选项就有用了。

### STRICT_TRANS_TABLES

在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做限制。（InnoDB默认事务表，MyISAM默认非事务表；MySQL事务表支持将批处理当做一个完整的任务统一提交或回滚，即对包含在事务中的多条语句要么全执行，要么全部不执行。非事务表则不支持此种操作，批处理中的语句如果遇到错误，在错误前的语句执行成功，之后的则不执行；MySQL事务表有表锁与行锁非事务表则只有表锁）

### NO_ZERO_IN_DATE
在严格模式下，不允许日期和月份为零

### NO_ZERO_DATE
设置该值，mysql数据库不允许插入零日期，插入零日期会抛出错误而不是警告。

### ERROR_FOR_DIVISION_BY_ZERO
在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如 果未给出该模式，那么数据被零除时MySQL返回NULL

### NO_AUTO_CREATE_USER
禁止GRANT创建密码为空的用户

### NO_ENGINE_SUBSTITUTION
如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常

### PIPES_AS_CONCAT
将”||”视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数Concat相类似

### ANSI_QUOTES
启用ANSI_QUOTES后，不能用双引号来引用字符串，因为它被解释为识别符

## 解决方案
有三种方式可以解决该问题。

### 关闭 ONLY_FULL_GROUP_BY
关闭 Mysql 的 ONLY_FULL_GROUP_BY 模式 又有两种方式。

方式一：通过以下命令关闭：
```shell
SET SESSION sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY,',''));
```

方式二：编辑`my.cnf`配置文件，可以通过以下命令查看配置文件所在目录：

```shell
mysql --help | grep cnf
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf
```

将 `ONLY_FULL_GROUP_BY` 关键字去掉：
```shell
[mysqld]
sql_mode = ""
```
然后重启Mysql 服务即可。

### ANY_VALUE
如果你不想更新配置文件，Mysql 还提供一种临时的解决方案——[ANY_VALUE()](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_any-value)。

使用 `ANY_VALUE()` 包裹的值不会被检查，跳过该错误。

```mysql
✅
select ANY_VALUE(a), ANY_VALUE(b), ANY_VALUE(c)
from table 
group by a,b; 
```

### 参考链接
* [记一次Group by 查询时的ONLY_FULL_GROUP_BY错误以及后续](https://blog.csdn.net/Abysscarry/article/details/79468411)
* [MySQL GROUP BY 的问题](https://www.cnblogs.com/Wayou/p/mysql_group_by_issue.html)