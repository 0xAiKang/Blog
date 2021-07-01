---
title: 记录 Laravel 开发中一些有意义的问题
date: 2021-06-28 22:31:56
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

如题。

<!-- more -->

## 问题一
为什么不要在生产环境使用 `composer update`？

其实这个问题和Laravel 并没有直接关系。

永远不要在生产环境上直接运行 `compser update`，因为它很慢并且会破坏版本库。

正确的做法应该是，始终在本地开发环境下使用 `composer update`，并将新的`composer.lock` 提交到版本库，生产环境中则需要运行 `composer install` 即可。

## 问题二
为什么这段代码永远都进不到 『用户注册失败』中？
```php
if($user){
   return '用户注册成功';
}

return '用户注册失败';
```

因为Laravel 的大部分操作，基本都是以异常形式处理，所以不需要 `if...else`。[原文链接](https://learnku.com/articles/25947#reply84579)。

## 问题三
具体哪些操作会触发模型事件？

实例一：
```php
$user = new User / find / first / all()->first();

// $user not exits; triggering event: creating、created、saving、saved
// $user exits; triggering event: updating、updated、saving、saved
$user->save(); 

$user->create();
$user->update();
$user->delete();
```
触发模型事件有一个很显著的特征就是：一定会存在模型实例。

实例二：
```php
User::where('id', 1)->update(['name', 'eienao']); 
```
上面这个例子，就不会触发模型事件，因为始终都没有一个模型实例参与。

实例三：
```php
$user = User::first();
$user->where('id', 1)->update(['name', 'eienao']); 
```
这个例子看起来可能会比较迷惑，但实际上最终也不会触发模型事件，因为 `where()` 方法返回的是一个『查询构造器』。

模型实例此时已经不参与其中了，其只是做一个引导出查询构造器的作用。

## 问题四
不要直接从 `.env` 文件中获取数据。

更好地做法应该是将数据放入配置文件，然后使用助手函数`config()` 去获取数据。

坏：
```php
$apiKey = env('API_KEY');
```

好：
```php
// config/api.php
'key' => env('API_KEY'),

// Use the data
$apiKey = config('api.key');
```

> 你可能会疑惑，为什么要这么做呢？最后不还是要从 `.env` 中获取数据。 

这是因为尽量不要修改业务代码，如遇变化要么修改`config`，要么修改`.env`，而`config` 的代码是纳入评审的，修改起来更方便，所以应该用`config` 代理`.env`, 能减少对`.env` 的修改。