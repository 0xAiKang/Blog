---
title: Laravel Carbon 快速上手
date: 2021-07-03 20:40:45
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

在使用PHP 开发时，免不了经常需要处理日期和时间，这使得我们不得不面临各种格式化、日期/时间计算等问题。

幸运的是，一些勤劳的人已经帮我们完成了辛苦的工作，使得在PHP 开发中处理日期/时间变得更加简单、更语义化。

<!-- more -->

## Carbon
[Carbon](https://github.com/briannesbitt/Carbon) 是由 [Brian Nesbit](https://github.com/briannesbitt) 开发的一个包，它扩展了 PHP 自己的 DateTime 类。

它提供了一些很好的功能来处理 PHP 中的日期/时间，诸如：
* 处理时区
* 日期解析
* 日期操作
* 日期比较
* 日期语义化
* 日期格式化
* 等

安装：
```php
composer require nesbot/carbon
```

使用：
```php
<?php
require __DIR__ . '/vendor/autoload.php';

use Carbon\Carbon;

echo Carbon::now();  // 2021-07-03 16:02:17
```
注意：因为Laravel 已默认安装了此包，所以无需再次执行上面的命令。

## 获取特定日期/时间

```php
// 获取当前时间 - 2021-07-03 16:03:46
echo Carbon::now();   // Object
echo new Carbon();

// 获取今天 - 2021-07-03 00:00:00
echo Carbon::today();

// 获取昨天 - 2021-07-02 00:00:00
echo Carbon::yesterday();

// 获取明天 - 2021-12-04 00:00:00
echo Carbon::tomorrow();

// 解析特定字符串 - 2021-01-01 00:00:00
echo new Carbon('first day of January 2021');

// 设定一个特定的时区 - 2021-01-01 00:00:00
echo new Carbon('first day of January 2021', 'Asia/Tokyo');
```

默认情况下，`Carbon` 的方法返回的为一个日期时间对象。
```php
Carbon {#179 ▼
  +"date": "2021-07-03 00:00:00.000000"
  +"timezone_type": 3
  +"timezone": "UTC"
}
```

虽然它是一个对象，但是却可以直接使用 `echo` 输出结果，因为有 `__toString`魔术方法。

## 日期格式化
如果你想把它转为字符串，可以使用 `toDateString` 或 `toDateTimeString`等方法：
```php
$dt = Carbon::now();

echo $dt->toDateString();               // 2021-07-03
echo $dt->toFormattedDateString();      // Jul 3, 2021
echo $dt->toTimeString();               // 16:40:01
echo $dt->toDateTimeString();           // 2021-07-03 16:40:01
echo $dt->toDayDateTimeString();        // Sat, Jul 3, 2021 4:40 PM
echo $dt->format('Y.m.d');              // 2021.07.03
```

## 日期解析
可以使用 `parse` 方法解析任何顺序和类型的日期。

```php
echo Carbon::parse("2021-07-03 16:20:44");
echo Carbon::parse("2021-07-03 16:20:44")->toDateString();
echo Carbon::parse("2021-07-03 16:20:44")->toDateTimeString();

echo Carbon::parse('now');        // 2021-07-03 16:24:01
echo Carbon::parse('today');      // 2021-07-03 00:00:00
echo Carbon::parse('yesterday');  // 2021-07-02 00:00:00
echo Carbon::parse('tomorrow');   // 2021-07-04 00:00:00

echo Carbon::parse('2 days ago'); // 2021-07-01 16:25:55 
echo Carbon::parse('+3 days');    // 2021-07-06 16:25:55
echo Carbon::parse('+3 weeks');   // 2021-07-24 16:25:55
echo Carbon::parse('+1 months');  // 2021-08-03 16:25:55
```

返回结果仍是 `Carbon` 类型的日期时间对象。

## 日期/时间操作
如何获取日期，并不是唯一需要做的事情，经常需要做的事情应该是操作日期或时间。

例如：计算一个日期加上N 天之后，是什么时间; 两个月后的今天是什么时间; 当前时间的三个小时之后是什么时间; 诸如此类。
```php
echo Carbon::now()->addDays(25);        // 2021-07-28 16:31:03
echo Carbon::now()->addWeeks(3);        // 2021-07-24 16:31:03
echo Carbon::now()->addHours(25);       // 2021-07-04 17:31:03
echo Carbon::now()->subHours(2);        // 2021-07-03 14:31:03  
echo Carbon::now()->addHours(2)->addMinutes(12; // 2021-07-03 18:43:03
echo Carbon::now()->modify('+15 days'); // 2021-07-18 16:31:03
echo Carbon::now()->modify('-2 days');  // 2021-07-01 16:31:03
```

## 日期比较
在 Carbon 中你可以使用下面的方法来比较日期：

* `min` –返回最小日期。
* `max` – 返回最大日期。
* `eq` – 判断两个日期是否相等。
* `gt` – 判断第一个日期是否比第二个日期大。
* `lt` – 判断第一个日期是否比第二个日期小。
* `gte` – 判断第一个日期是否大于等于第二个日期。
* `lte` – 判断第一个日期是否小于等于第二个日期。

```php
$now = Carbon::now();
$yesterday = Carbon::yesterday();

var_dump($now->eq($yesterday));     // bool(false)
var_dump($now->gt($yesterday));     // bool(true)
var_dump($now->lt($yesterday));     // bool(false)
```

## 语义化
相对时间语义化变得越来越流行，通常可以在各种社交、通讯应用上看到。

例如，将时间显示为 3 小时前 比显示 上午 8:12，更适合人类阅读。

这些方法被用于计算时间差，并转换为人类可阅读的格式，有如下四种表达时间差的方式：
* 将一个过去的时间和现在做比较：
  * 1 小时前
  * 5 个月前
* 将一个未来的时间和现在做比较：
  * 1 小时后
  * 5 个月后
* 将一个过去的时间和另一个时间做比较：
  * 1 小时前
  * 5 小时前
* 将一个未来的时间和另一个做比较：
  * 1 小时后
  * 5 小时后

```php
$dt = Carbon::now();

echo $dt->subDays(10)->diffForHumans();   // 1 week ago
echo $dt->addHours(12)->diffForHumans();  // 1 week ago
echo $dt->subMonth()->diffForHumans();    // 1 month ago
```

## 本地化
上面最后一个例子，可以看到，Carbon 默认输出的不是中文，可以增加以下代码设置本地化。

```php
\Carbon\Carbon::setLocale('zh');

$dt = Carbon::now();

echo $dt->subDays(10)->diffForHumans();   // 1周前
echo $dt->addHours(12)->diffForHumans();  // 1周前
echo $dt->subMonth()->diffForHumans();    // 1个月前
```

Carbon 能做的远远不止这些，这里只是列举了一些个人常用的方法，关于更多Carbon 的用法，请查看[官方文档](https://carbon.nesbot.com)。

## 最佳实践
下面整理一些常见的使用场景。

获取某个时刻的起始时间和结束时间：
```php
// 获取一天的开始时间和结束时间
now()->startOfDay();      // 2021-08-12 00:00:00
now()->endOfDay();        // 2021-08-12 23:59:59

// 获取这周的开始时间和结束时间
now()->startOfWeek();     // 2021-08-09 00:00:00
now()->endOfWeek();       // 2021-08-15 23:59:59

// 获取这个月的起始时间和结束时间
now()->startOfMonth();    // 2021-08-01 00:00:00
now()->endOfMonth();      // 2021-08-31 23:59:59

// 获取今年的起始时间和结束时间
now()->startOfYear();     // 2021-01-01 00:00:00
now()->endOfYear();       // 2021-12-31 23:59:59
```

获取指定日期范围内的日期：
```php
$startTime = now();
$endTime = Carbon::parse("2021-09-12");
$dates = $startTime->daysUntil($endTime);
```

格式化日期为指定格式：
```php
now()->format("Y/m/d");   // 2021/08/12
now()->format("Y.m.d");   // 2021.08.12
```

判断日期是否为指定格式：
```php
Carbon::hasFormat("2021/08/12", "Y-m-d")  // false
```

## 参考链接
* [Laravel 中日期时间 Carbon 包的的使用详解](https://www.heibaiketang.com/forum/show/141.html)
* [Carbon Api Doc](https://carbon.nesbot.com/docs/)