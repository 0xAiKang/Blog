---
title: 浅谈 Mysql 事务与锁
date: 2021-07-18 11:12:26
tags: ["Mysql"]
categories: ["Mysql"]
---

事务是Mysql InnoDB 引擎的一个重要特点，具有ACID 四个特性。

<!-- more -->

* 原子性（Atomicity）：事务的所有操作，要么全部完成，要么全部不完成，不会结束在某个中间环节。
* 一致性（Consistency）：事务开始之前和事务结束之后，数据库的完整性限制未被破坏。
* 隔离性（Isolation）：两个或者多个事务的执行是互不干扰的，一个事务不可能看到其他事务运行时，中间某一时刻的数据。
* 持久性（Durability）：事务完成之后，事务所做的修改进行持久化保存，不会丢失。

平时项目和工作中，会很频繁使用到事务，但是，一些细节如果不稍加注意，是很容易出现问题的。

## 观察事务

### 示例一

这是一段很常见的代码，逻辑也很简单，首先开启事务，如果`try` 代码块没有异常，提交事务; 如果`try` 代码块遇到异常，事务进行回滚。

```php
Route::get("test", function () {
    DB::beginTransaction();
    try {
        \App\Models\User\UserModel::whereUid(7)
            ->update([
                "balance" => "1000",
            ]);
        DB::commit();
    } catch (\Exception $exception) {
        DB::rollBack();
        throw new Exception("操作失败");
    }

    return "success";
});
```

在这个过程中，我们只知道，手动选择了开启、提交或者回滚事务，但是对于事务执行的整个过程，比如：什么时候开启了事务、什么时候提交的事务、我们都是毫无感知的。

那有没有什么办法，可以看到整个过程呢？

答案是有的。

### 示例二

在新的代码示例中，只加了一行代码，它的作用是**延缓事务提交**（这里 sleep 15秒，便于观察）。
```php
Route::get("sleep15", function () {
    DB::beginTransaction();
    try {
        \App\Models\User\UserAccountModel::whereUid(7)
            ->update([
                "balance" => "2000",
            ]);
        sleep(15);
        DB::commit();
    } catch (Exception $exception) {
        DB::rollBack();
        throw new Exception("操作失败");
    }

    return "success";
});
```

同时需要配合Mysql 客户端，执行一个SQL 语句，查看正在进行中的事务：
```bash
SELECT * FROM information_schema.INNODB_TRX;
```

可以在发送请求之前先执行一次：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718105708.png)

发送请求之后执行一次：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718105734.png)

请求结束之后再执行一次：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718105708.png)

可以很清晰地看到，事务从无到有再到无的整个过程：
1. 发送请求之前，此时事务还没有开启。
2. 发送请求之后，此时事务已开启，但因为sleep 的原因，没法直接提交或者回滚，只能一直开启事务等待。
3. sleep 结束，此时事务进行提交或回滚，请求结束。

往往因为事务使用不当，而造成锁表等问题，原因大多出在了第二步上。

## 观察锁
Mysql 的锁（这篇笔记就不对锁的分类具体展开说明了），往往都是伴随事务出现。

为了演示『锁』是如何产生的，这次需要同时用到上面的两个示例。

首先请求`127.0.0.1:8000/sleep15`，在请求结束之前，请求`127.0.0.1:8000/test` 。

此时观察请求状态，可以发现两个接口都没有马上响应。

再次打开Mysql 客户端，查看当前正在进行中的事务：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718105913.png)

不出意外，可以发现此时等待的事务变成了两个。

通过`trx_id` 大小，可以判断出，先请求的`127.0.0.1:8000/sleep15` 事务当前正在运行中，而后面请求的`127.0.0.1:8000/test` 事务，则是被锁住等待，等待前面的事务释放（提交或者回滚）。

同理，如果此时请求不是两个，而是多个，相应的，被锁住的事务就是多个。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718105924.png)

## 结语

这个问题看着挺简单的，但实际开发时，往往容易被忽略。

过早开启事务，提交或者回滚事务之前，穿插许多其他业务逻辑，如果其他某个逻辑超时，则会导致事务不能及时释放，从而出现连锁反应。