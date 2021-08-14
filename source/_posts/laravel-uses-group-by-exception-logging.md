---
title: Laravel Group By 异常记录
date: 2021-08-14 16:16:53
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

前段时间用Laravel 做项目，使用Group By时，总是会遇到的一个错误，具体异常信息如下：

<!-- more -->

> Illuminate\Database\QueryException: SQLSTATE[42000]: Syntax error or access violation: 1140 In aggregated query without GROUP BY, expression #2 of SELECT list contains nonaggregated column 'database.table.id'; this is incompatible with sql_mode=only_full_group_by 

这个错误很眼熟，其原因是对于聚合操作（如：sum、max、min等），如果在SELECT 中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中。

简而言之，就是SELECT 后面接的列必须被GROUP BY 后面接的列所包含。

可是同样的SQL，在数据库中是可以正常执行的，为什么在Laravel 下却总是会报错呢？

当时也是因为项目紧急，没具体深究其原因。

今天无意间看到一篇文章——[Laravel使用group by报错的问题](http://www.manongjc.com/detail/24-naczktadrzjmysh.html)，正好把这个问题给讲明白了。

原来是因为开发者在Laravel `5.3` 版本后增加一个数据库的`strict` 模式，其中一个开发者对于增加这个模式的看法很有意思：

> Adam14Four ：
To be completely honest, I don't remember exactly what the details were, but it was some sort of data-loss problem.
说真的，我也忘了具体的细节了，可能是因为数据丢失排序的问题。

Laravel `8.x` 版本，默认会启用该模式，启用时，会造成以下影响：
> fernandobandeira：
1 - Add all columns to group by.
group by需要所有的列。
2 - Won't be able to use date's such as 0000-00-00 00:00:00.
时间不能使用0000-00-00 00:00:00的数据。
3 - Fields like boolean will throw fatal if you pass something that isn't a boolean value, like 1, before it would convert it to true and save, with strict it fails.
字段如果是boolean类型，但是传入一个非boolean如「1」将会抛出一个致命错误。在非strict模式下会自动转换成true并保存。
4 - You'll get an error if you divide a field by 0 (or another field that has 0 as value).
如果一个字段除以0将会得到一个错误（或者其他有0值的字段）

## 解决方案
**1. 最简单的方案，关闭该模式即可：**

编辑`/config/database.php`：
```php
'connections' => [
    'mysql' => [
        // ...
        'strict' => false,
    ],
],
```

**2. 配置modes**

编辑`/config/database.php`：
```php
'connections' => [
    'mysql' => [
        // ...
        'modes' => [
            'ONLY_FULL_GROUP_BY',
            'STRICT_TRANS_TABLES',
            'NO_ZERO_IN_DATE',
            'NO_ZERO_DATE',
            'ERROR_FOR_DIVISION_BY_ZERO',
            'NO_AUTO_CREATE_USER',
            'NO_ENGINE_SUBSTITUTION',
        ],
    ],
],
```

不知为何，验证此方案时总是未生效。

**3. full group by**
如果不想关闭该模式，那就只有两种选择了：
1. Full Group By：SELECT 查询的字段包含所有的Group By 的字段中
2. ANY_VALUE：这是一个Mysql 的函数，使用它可以临时跳过一些错误

但需要注意的是，这种方案并不会“一劳永逸”，以后的每一次Group By 还是会面临相同的问题。

## 参考链接
* [Laravel使用group by报错的问题](http://www.manongjc.com/detail/24-naczktadrzjmysh.html)
* [query that worked in Laravel 5.2 gives me error in Laravel 5.3](https://github.com/laravel/framework/issues/14997)