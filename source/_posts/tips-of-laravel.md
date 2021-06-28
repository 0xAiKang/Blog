---
title: Tips of Laravel
date: 2021-06-20 15:00:12
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

Awesome tips for Laravel.

<!-- more -->

## 善用集合
Collections 是 Laravel 提供的一个巨大特性，它允许我们轻松地操作数组，可以为我们节省大量时间。

比如想要对下面这组数据进行求和：

```php
$orders = [
    [
        "id" => 1000,
        "price" => 80,
    ],
    [
        "id" => 1001,
        "price" => 120,
    ],
    [
        "id" => 1002,
        "price" => 30,
    ],
];
```

使用传统的 `foreach` 方式：
```php
$total_price = 0;
foreach ($orders as $order) {
    $total_price += $order["price"];
}
```

试试使用集合：
```php
$total_price = collect($orders)->pluck("price")->sum();
```

虽然两种方式都可以实现，但显然使用集合更容易一些，更多集合的最佳实践可以查看我的另一篇笔记——[Laravel Collection 实际使用](https://www.0x2beace.com/the-actual-use-of-collection-in-laravel)。

善用集合，可以帮我们减少很多重复的代码。

## 查询作用域
通常，在Laravel Eloquent ORM 查询时，需要匹配某些条件时，一般会这样写：

```php
$admin = Admin::where("is_enable", true)
    ->where("is_admin", true)
    ->get();
```

这样写并没有什么问题，但为了使我们的代码更具可读性，而不是重复性，可以使用 `query scope`，在对应模型中创建查询作用域：

```php
public function scopeEnable($query)
{
    return $query->where('is_enable', true);
}

public function scopeAdmin($query)
{
    return $query->where('is_admin', true);
}
```

现在，可以通过如下方式进行查询：
```php
$admin = User::enable()
    ->admin()
    ->get();
```

-------

如果某个查询条件频繁使用到了，可以在模型中添加全局查询作用域，这样可以默认加上该查询条件：
```php
protected static function booted()
{
    static::addGlobalScope("is_deleted", function (Builder $builder) {
        $builder->where("is_deleted", false);
    });
}
```

> 取消全局查询作用域？

```php
// 指定类
User::withoutGlobalScope(EmailVerifiedAtScope::class)->get();

// 指定字段
User::withoutGlobalScope('is_deleted')->get();

// 移除所有全局作用域
User::withoutGlobalScopes()->get();

// 移除多个类/匿名函数
User::withoutGlobalScopes([FirstScope::class, SecondScope::class])->get();
```

## Eloqunt Query
实际开发中，因为需求的复杂性，我们往往需要写出各种各样的SQL 来满足查询。

`selectRaw()`、`whereRaw()`、`havingRaw()` 允许我们在查询构造器中，加入原始SQL 查询，例如，统计分组数量：

```php
$count = User::groupBy("is_enable")
    ->selectRaw("count(id) as aggregate")
    ->get();
```

## Log and Debug
Laravel 为我们提供了便捷的调试代码方式——`dd()`，但某些场景下并不适合使用 `dd()`，比如测试回调是否正常。

这时可以使用 `Log` 助手函数进行调试，生成的日志在`storage/logs` 目录下。

```php
\Log::debug('Test Message', $result]);
```

`dd()` 作为现代开发者的调试利器，日常开发基本上离不开它，也许你一直都是这么用的：

```php
$users = User::where('name', 'Taylor')->get();
dd($users);
```

其实有一种更简单的方式：
```php
$users = User::where('name', 'Taylor')->get()->dd();
```
它可以作为一个链式方法，直接放在 Eloquent Query 或者集合的后面进行调用。

## Tinker
Laravel 的另一大特性就是提供了交互式的命令行——Tinker，在这里你可以执行各种代码，而无需考虑环境，在某些时候，进行调试时是极为方便的。

```sh
php artisan tinker
```

我通常会使用 `Tinker` 做以下事情：
* 检测某段代码是否符合预期
* Eloquent Query 测试
* SDK 测试

## 分页求和
在有分页的情况下，如何统计某个字段所有记录的总和？

```php
// 创建一个查询构造器
$query = Post::query();

// 在查询分页之前求和
$sum = $query->sum('post_views');

// 查询分页
$posts = $query->paginate(10);
```

## Data Get Function
如果有一个复杂的数组对象数据结构，可以使用 `data_get` 助手函数使用`.` 表示法和 `*` 通配符从嵌套数组或对象中检索值：

```php
$data = [
    0 => ['user_id' =>'1',  'post' => ["id" => 1000],],
    1 => ['user_id' =>'2',  'post' => ["id" => 1001], ],
    2 => ['user_id' =>'3',  'post' => ["id" => 1002], ],
];

$ids = data_get($data, "*.post.id");
// [1000, 1001, 1002]
```

## optional
`optional()` 方法允许你获取对象的属性时调用该方法。如果该对象为 null，那么属性或者方法也会返回 null 而不是引起一个错误：

```php
// User 2 exists, without account
$user2 = User::find(2);
$accountId = $user2->account->id; // PHP Error: Trying to get property of non-object

// Fix without optional()
$accountId = $user2->account ? $user2->account->id : null; // null
$accountId = $user2->account->id ?? null; // null

// Fix with optional()
$accountId = optional($user2->account)->id; // null
```

## 封装SDK
通常在安装了一个 SDK 之后，我们可以做一些简单的封装，这样使用起来会更方便。

```bash
php artisan make:provider JpushServiceProvider
```

这里以极光推送 这个第三方推送服务商为例：
```php
<?php

namespace App\Providers;

use JPush\Client;
use Illuminate\Support\ServiceProvider;

class JpushServiceProvider extends ServiceProvider
{
    public function boot()
    {
        //
    }

    public function register()
    {
        $this->app->singleton(Client::class, function ($app) {
            return new Client(config('jpush.key'), config('jpush.secret'));
        });

        $this->app->alias(Client::class, 'jpush');
    }
}
```

加入到 `config/app.php`：
```php
'providers' => [
  App\Providers\JpushServiceProvider::class,
]
```

创建配置文件：
```php
<?php

return [
    'key' => env('JPUSH_KEY'),
    'secret' => env('JPUSH_SECRET'),
];
```

在 env 文件中填写 Jpush 的 key 和 secret：
```
# jpush
JPUSH_KEY=9c6f53edad67db7ec24bfe32
JPUSH_SECRET=deeb2a04669ab79******
```

这样我们可以直接依赖注入 `JPush\Client` 或者 `app('jpush')` 来使用 Jpush 的 SDK。