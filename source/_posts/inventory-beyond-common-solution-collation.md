---
title: 库存超出常见解决方案整理
date: 2022-04-16 17:46:18
tags: ["PHP"]
categories: ["PHP"]
---

> 原文链接：[使用laravel解决库存超出的几个方案](https://learnku.com/articles/57959)

库存超出是一个常见的幂等问题，下面介绍一下解决超卖问题常见的一些方案
*  Redis 存储库存
*  Redis 原子锁
*  Mysql 悲观锁 
*  Mysql 乐观锁

<!-- more -->

## 准备

准备一张实验表：
```mysql
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(11) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | varchar(10)      | YES  |     | NULL    |                |
| num   | int(11)          | YES  |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
```

使用 `go` 模拟并发：
```go
package main

import (
    "fmt"
    "github.com/PeterYangs/tools/http"
    "sync"
)

func main() {
    client := http.Client()
    wait := sync.WaitGroup{}
    for i := 0; i < 50; i++ {
        wait.Add(1)
        go func(w *sync.WaitGroup) {
            defer w.Done()
            res, _ := client.Request().GetToString("http://www.api/test1?id=1")
            fmt.Println(res)
        }(&wait)
    }
    wait.Wait()
}
```

## 错误示范

```php

function test()
{

    $id = request()->input('id');
    $product = Product::where('id', $id)->firstOrFail();

    if ($product->num <= 0) {
        return "卖光啦！！";
    }

    $product->decrement('num');
    return "success";
}
```

查看库存：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220416165310.png)
库存超出。

## Redis 存储库存

```php
public function test()
{
    $id = request()->input('id');
    $redis = new \Redis();
    $num = $redis->rawCommand('get', 'product_' . $id);
    if ($num <= 0) {
        return "卖完啦！";
    }

    //减库存
    $result = $redis->rawCommand('decrby', 'product_' . $id, 1);
    //减多了回滚
    if ($result < 0) {
        $redis->rawCommand('incrby', 'product_' . $id, 1);
        return "卖完啦！";
    }
    return 'success';
}
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220416172328.png)

库存正常。

## Redis 原子锁

```php
public function test()
{
    $id = request()->input('id');
    $lock = \Cache::lock("product_" . $id, 10);
    try {

        //最多等待5秒，5秒后未获取到锁，则抛出异常
        $lock->block(5);
        $product = TbModel::where('id', $id)->firstOrFail();

        if ($product->num <= 0) {
            return "卖光啦！！";
        }

        $product->decrement('num');
        return 'success';
    }catch (LockTimeoutException $e) {
        return '当前人数过多';
    } finally {
        optional($lock)->release();
    }
}
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220416165550.png)
库存正常。

## Mysql 悲观锁

```php
public function test()
{
    $id = request()->input('id');

    try {
        \DB::beginTransaction();
        $product = Product::where('id', $id)->lockForUpdate()->first();

        if ($product->num <= 0) {
            return "卖光啦！！";
        }

        $product->decrement('num');
        \DB::commit();
        return "success";
    } catch (\Exception $exception) {
        return "error";
    }
}
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220416165918.png)

库存正常。

## Mysql 乐观锁

```php
public function test()
{
    $id = request()->input('id');
    $product = TbModel::where('id', $id)->first();
    if ($product->num <= 0) {
        return "卖光啦！！";
    }

    //修改前检查库存和之前是否一致，不一致说明已经有变动，则放弃更改
    $res = \DB::update('UPDATE `tb` SET num = num -1 WHERE id = ? AND num=?', [$id, $product->num]);
    if (!$res) {
        return '当前人数过多';
    }
  
    return 'success';
}
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220416165918.png)

库存正常。

优化乐观锁，修改库存的 sql 修改为：
```sql
\DB::update('UPDATE `tb` SET num = num -1 WHERE id = ? AND num-1 >= 0', [$id]);
```

## 总结
以上几种方案都可以有效解决库存超出的问题，应用时可以根据实际具体场景进行选择，优先考虑顺序为：\
Redis 存储 > Redis 原子锁 > Mysql 悲观锁/乐观锁

## 参考链接
* [使用laravel解决库存超出的几个方案](https://learnku.com/articles/57959)