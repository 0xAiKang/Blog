---
title: PHP 8.0 初体验
date: 2020-12-01 21:09:15
tags: ["PHP"]
categories: ["PHP"]
---

昨天使用 homebrew 安装软件时，结果把我本地已安装的软件中能更新的全部给更新了一遍。

这其中就包括 `php8.0`。在`8.0` 正式出来之前，有听说过加入了新特性：JIT编译。

从理论上讲，JIT处理PHP脚本编译的方式能够提高应用程序的速度，但究竟能有多快呢？下面通过一个简单的例子来看看。

```
<?php
$startTime = microtime(true);
$mysqli = new Mysqli("127.0.0.1", "root", "root");

function doSomething($db,$i)
{
	$hash = md5($i);
	$db->query("INSERT INTO local.test(id, hash) VALUES($i, \"$hash\")");
}

$i = 1;
while ($i<100000) {
	doSomething($mysqli, $i);
	$i++;
}

$total = microtime(true) - $startTime;
var_dump("总耗时：{$total}秒");
```
这里只是简单的向数据库不重复插入十万条数据。
我知道用这个脚本举例子并不好，但它却是离我日常使用最近的。

`php7.3` 测试结果：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201201000322.png)

`php8.0` 未开启 JIT 扩展测试结果：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201201000427.png)

`php8.0` 已开启 JIT 扩展测试结果：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201201000725.png)

可以看到相比 7.3，足足快了近三分之一！

当然这个测试结果严格意义上来讲，并不准确，但看到数字从四十多秒缩短到三十秒，还是很惊喜的。

我的电脑配置：
* 3.5 GHz 双核Intel Core i7
* 16 GB RAM