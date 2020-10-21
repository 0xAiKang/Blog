---
title: PHP 常见浮点数操作
date: 2020-10-20 19:24:23
tags: ["PHP"]
categories: ["PHP"]
---

浮点数操作在实际应用中还是挺多的，这篇笔记用来整理常见操作。

<!-- more -->

## 保留N位小数做四舍五入
想要保留N 位小数同时做四舍五入的方式还是挺多的，下面列举常用的几种。

### sprintf
[sprintf](https://www.php.net/manual/zh/function.sprintf.php) 函数用于返回一个格式化之后的字符串。

```
<?php
$num = 22.356;
echo sprintf("%.2f", $num); // 22.36
```
`%.2f` 是目标格式，其中2 表示2 位，`f`表示视为浮点数。

### round
[round](https://www.php.net/manual/zh/function.round) 函数用于对浮点数进行四舍五入。

还可以通过传入参数，决定从第几位开始四舍五入。如果没有参数，默认从小数点后一位开始四舍五入。

```
<?php
echo round(3.4);         // 3
echo round(3.5);         // 4
echo round(22.356, 2);   // 22.36
```

## 保留N位小数不做四舍五入

```
<?php
$num = 22.356;
echo sprintf("%.2f",substr(sprintf("%.3f", $num), 0, -1));  // 22.35
```

## 获取小数位长度

```
<?php
$num = 22.356;
echo strlen(substr(strrchr($num, "."), 1));  // 3
```
