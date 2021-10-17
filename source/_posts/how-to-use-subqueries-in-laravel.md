---
title: 如何在 Laravel 中使用子查询
date: 2021-10-17 19:16:38
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

这里并不讨论子查询的效率问题，而是单就如何在Laravel 中使用子查询展开讨论。

<!-- more -->

使用子查询时，必须遵循以下规则：
* 子查询必须括在圆括号中
* 子查询的 SELECT 子句中只能有一个列，除非主查询中有多个列，用于与子查询选中的列相比较
* 子查询不能使用 ORDER BY，不过主查询可以。在子查询中，GROUP BY 可以起到同 ORDER BY 相同的作用
* 返回多行数据的子查询只能同多值操作符一起使用，比如 IN 操作符
* SELECT 列表中不能包含任何对 BLOB、ARRAY、CLOB 或者 NCLOB 类型值的引用子查询不能直接用在集合函数中
* BETWEEN 操作符不能同子查询一起使用，但是 BETWEEN 操作符可以用在子查询中

在Laravel 中创建子查询有两种方式：
1. 构建 raw 语句
2. 使用查询构造器的闭包

### 方式一
```php
$sub = UserAccountModel::selectRaw("max(balance) as balance");
$result = UserAccountModel::whereRaw("balance = ({$sub->toSql()})")
    ->mergeBindings($sub->getQuery())
    ->first();
```

这里用到了几个API：
**1. `toSql()` 获取不带 binding 参数的 SQL 语句（通常是带问号的SQL）：**
```
"select max(balance) as balance from `user_account`"
```

**2. `getQuery()` 获取 binding 参数：**
```php
Illuminate\Database\Query\Builder {#1015 ▼
  +connection: Illuminate\Database\MySqlConnection {#982 ▶}
  +grammar: Illuminate\Database\Query\Grammars\MySqlGrammar {#983 ▶}
  +processor: Illuminate\Database\Query\Processors\MySqlProcessor {#984}
  +bindings: array:9 [▶]
  +aggregate: null
  +columns: array:1 [▶]
  +distinct: false
  +from: "user_account"
  +joins: null
  +wheres: []
  +groups: null
  +havings: null
  +orders: null
  +limit: null
  +offset: null
  +unions: null
  +unionLimit: null
  +unionOffset: null
  +unionOrders: null
  +lock: null
  +beforeQueryCallbacks: []
  +operators: array:30 [▶]
  +useWritePdo: false
}
```

**3. `mergeBindings()` 将 binding 参数合并到查询中**

最终获得SQL 如下：
```
select * from `user_account` where `balance` = (select max(balance) as balance from `user_account` limit 1) limit 1
```

### 方式二
直接使用查询构造器自带的闭包查询：
```php
$result = UserAccountModel::where("balance", function ($query){
    $query->selectRaw("max(balance) as balance")
        ->from("user_account")
        ->value("balance");
})->first();
```

最终获得SQL 如下：
```php
select * from `user_account` where `balance` = (select max(balance) as balance from `user_account` limit 1) limit 1
```