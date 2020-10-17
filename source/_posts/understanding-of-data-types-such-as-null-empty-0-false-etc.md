---
title: 对于NULL、空、0、false等数据类型的理解
date: 2020-10-17 17:21:51
tags: ["PHP"]
categories: ["PHP"]
---

之所以决定写这片笔记，是因为一直对 空 这个概念很模糊，在代码逻辑中常会遇到需要判断的时候，总是模拟两可。

<!-- more -->

常见的“空”有以下这些：
* 整形0：0
* 字符1：1
* 字符空：""
* 字符零："0"
* 空数组：[]
* true
* false
* null

NUll 上面的那些都好理解，都是常见的，重点介绍一下NULL。

### NULL 是什么？

> Null是在计算机具有保留的值，可以用于指针不去引用对象，现在很多程序都会使用指针来表示条件，但是在不同的语言中，含义是不一样的。

这里我们只介绍 PHP 中的 NULL。

在 PHP 中，表示一个变量没有赋值、或者是被赋值的值为 NULL，以及被 unset 的。


### 使用PHP 函数对变量进行比较：

| 表达式          | [gettype()](https://www.php.net/manual/zh/function.gettype.php) | [empty()](https://www.php.net/manual/zh/function.empty.php) | [is_null()](https://www.php.net/manual/zh/function.is-null.php) | [isset()](https://www.php.net/manual/zh/function.isset.php) | [boolean](https://www.php.net/manual/zh/language.types.boolean.php) : `if($x)` |
| :-------------- | :----------------------------------------------------------- | :---------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------------- | :----------------------------------------------------------- |
| `$x = "";`      | [string](https://www.php.net/manual/zh/language.types.string.php) | **`TRUE`**                                                  | **`FALSE`**                                                  | **`TRUE`**                                                  | **`FALSE`**                                                  |
| `$x = null;`    | [NULL](https://www.php.net/manual/zh/language.types.null.php) | **`TRUE`**                                                  | **`TRUE`**                                                   | **`FALSE`**                                                 | **`FALSE`**                                                  |
| `var $x;`       | [NULL](https://www.php.net/manual/zh/language.types.null.php) | **`TRUE`**                                                  | **`TRUE`**                                                   | **`FALSE`**                                                 | **`FALSE`**                                                  |
| $x is undefined | [NULL](https://www.php.net/manual/zh/language.types.null.php) | **`TRUE`**                                                  | **`TRUE`**                                                   | **`FALSE`**                                                 | **`FALSE`**                                                  |
| `$x = array();` | [array](https://www.php.net/manual/zh/language.types.array.php) | **`TRUE`**                                                  | **`FALSE`**                                                  | **`TRUE`**                                                  | **`FALSE`**                                                  |
| `$x = false;`   | [boolean](https://www.php.net/manual/zh/language.types.boolean.php) | **`TRUE`**                                                  | **`FALSE`**                                                  | **`TRUE`**                                                  | **`FALSE`**                                                  |
| `$x = true;`    | [boolean](https://www.php.net/manual/zh/language.types.boolean.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = 1;`       | [integer](https://www.php.net/manual/zh/language.types.integer.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = 42;`      | [integer](https://www.php.net/manual/zh/language.types.integer.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = 0;`       | [integer](https://www.php.net/manual/zh/language.types.integer.php) | **`TRUE`**                                                  | **`FALSE`**                                                  | **`TRUE`**                                                  | **`FALSE`**                                                  |
| `$x = -1;`      | [integer](https://www.php.net/manual/zh/language.types.integer.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = "1";`     | [string](https://www.php.net/manual/zh/language.types.string.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = "0";`     | [string](https://www.php.net/manual/zh/language.types.string.php) | **`TRUE`**                                                  | **`FALSE`**                                                  | **`TRUE`**                                                  | **`FALSE`**                                                  |
| `$x = "-1";`    | [string](https://www.php.net/manual/zh/language.types.string.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = "php";`   | [string](https://www.php.net/manual/zh/language.types.string.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = "true";`  | [string](https://www.php.net/manual/zh/language.types.string.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |
| `$x = "false";` | [string](https://www.php.net/manual/zh/language.types.string.php) | **`FALSE`**                                                 | **`FALSE`**                                                  | **`TRUE`**                                                  | **`TRUE`**                                                   |


### 松散判断 ==
|             | **`TRUE`**  | **`FALSE`** | `1`         | `0`         | `-1`        | `"1"`       | `"0"`       | `"-1"`      | **`NULL`**  | `array()`   | `"php"`     | `""`        |
| :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | ----------- |
| **`TRUE`**  | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** |
| **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`TRUE`**  | **`FALSE`** | **`TRUE`**  |
| `1`         | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `0`         | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`TRUE`**  |
| `-1`        | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `"1"`       | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `"0"`       | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `"-1"`      | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| **`NULL`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`TRUE`**  | **`FALSE`** | **`TRUE`**  |
| `array()`   | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`TRUE`**  | **`FALSE`** | **`FALSE`** |
| `"php"`     | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** |
| `""`        | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`TRUE`**  |


### 严格比较 ===
|             | **`TRUE`**  | **`FALSE`** | `1`         | `0`         | `-1`        | `"1"`       | `"0"`       | `"-1"`      | **`NULL`**  | `array()`   | `"php"`     | `""`        |
| :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | :---------- | ----------- |
| **`TRUE`**  | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `1`         | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `0`         | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `-1`        | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `"1"`       | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `"0"`       | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `"-1"`      | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| **`NULL`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** | **`FALSE`** |
| `array()`   | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** | **`FALSE`** |
| `"php"`     | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  | **`FALSE`** |
| `""`        | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`FALSE`** | **`TRUE`**  |

### 参考链接
[PHP 类型比较表](https://www.php.net/manual/zh/types.comparisons.php)