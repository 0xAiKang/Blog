---
title: Laravel Eloquent ORM 常用操作整理
date: 2021-04-04 09:22:30
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

Laravel 支持原生的 SQL 查询、流畅的查询构造器 和 Eloquent ORM 三种查询方式：

<!-- more -->

* 流畅的查询构造器（简称DB），它是为创建和运行数据库查询提供的一个接口，支持大部分数据库操作，和手写SQL 的本质是一样的。
* Eloquent ORM（简称ORM），是一个对象关系映射(Object Relational Mapper)工具，通过建立模型与数据表进行交互，它会把数据库中的数据映射成对象和集合对象，无需接触底层数据，可以直接调用映射出来的对象进行开发。

这篇笔记主要来整理下常用的ORM 操作。

## 查询
`artisan tinker` 是 Laravel 框架自带的命令，用以调出 Laravel 的交互式运行时，Eloquent ORM 的代码可以直接在该环境中运行。

### 查询列表

获取所有数据：
```php
use App\Models\User;
$users = User::all();
```

如果只需要部分字段，有两种方式进行限定：
```php
$users = User::all(["id", "name"]);

$users = User::select("id", "name")->get();
```

获取单列：
```php
$name = User::pluck('name');
// ["boo", "mac", "yumi"]
```

还可以在返回的集合中指定字段的自定义键名，注意：该自定义键必须是该表的其它字段列名，否则会报错：
```php
$name = User::pluck('email','name');
// ["boo" => "boo@example.com", "yumi" => "yumi@example.com"]
```

### 查询单条数据
```php
// 通过主键获取模型
$user = User:;find(1);

// 获取匹配查询条件的第一个模型
$user = User::where('is_enable', 1)->first();

// 获取第一条数据的指定列值
$user = User::value("name");  
// 返回结果是字符串：boo

// 传递主键数组来调用 find 方法，返回匹配记录集合
$user = User::find([1,2,3]);  
// 等同于 
$user = User::whereIn("id", [1,2,3])->get();
```

### 处理返回结果集
Eloquent ORM 查询返回值是 `Illuminate\Database\EloquentCollection` 的一个实例，所以除了可以使用传统的数组方式进行遍历，还可以使用集合方式进行遍历。

#### chunk
`chunk`方法可以把大的结果集分成小块查询，例如，我们可以将全部User 表数据切割成一次处理 `5` 条记录的一小块：
```php
$result = User::chunk(5, function ($users) {
    foreach ($users as $user) {
        echo $user->name.PHP_EOL;
    }
});
// result 为 boolean 
```

在User表中一共有`14`条数据，通过查看查询日志，可以看到`chunk` 分了三次查询 ：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210403172906.png)

#### each
如果想对一个集合中的每一项都进行一些操作，但不修改集合本身，则可以使用`each`：
```php
$users = User::all();
$users = $users->each(function ($user , $key) {
    $user->password = bcrypt(122410);
});
// 返回结果包含完整的User模型，其中password 字段的值被修改
```

#### map

如果想对集合中的所有元素进行迭代，对它们进行修改，并返回包含修改的新集合，那么需要使用`map`：
```php
$users = User::all();
$users = $users->map(function ($user, $key) {
    return [
        "name" => $user->name,
        "password" => bcrypt(122410),
    ];
});
// 返回结果仅包含name 和password 字段，其中password 字段的值被修改
```

### 聚合方法

```php
// 统计总数
$count = User::count();

// 统计分组
$count = User::groupBy("is_enable")->selectRaw("count(id) as aggregate")->get();
// 注意不能这样写：User::select('count(id) as aggregate')->groupBy("is_enable")->get();
```

### 条件查询

构建复杂查询：
```php
// 组合查询方式一
$where = [];
$where[] = ["is_enable", 1];
$where[] = [
  function($query){
  $query->where("id", ">", 10)
    ->orWhere("name", "like", "%admin%");
}];
User::select("id", "name as username", "email")->where($where)->get(); 

// 组合查询方式二
$builder = User::select("id", "name as username", "email");
$builder->where("is_enable", 1);
$builder->where(function ($query){
  $query->where("id", ">", 10)
    ->orWhere("name", "like", "%admin%");
});
$users = $builder->get();

// 两种方式的查询SQL 是一样的： select `id`, `name` as `username`, `email` from `users` where (`is_enable` = '1' and (`id` > '10' or `name` like '%admin%'))

// Where Exists
$builder = User::select("id", "name", "email");
$builder->whereExists(function ($query){
    $query->select(User::raw(1))
        ->from("topics")
        ->whereRaw("topics.user_id = users.id");
});
// 查询发过文章的用户
```

### 排序
```php
// 用户id 倒序
$user = User::orderBy("id", "desc")->get();

// 获取created_at 最大的一条记录
$user = User::latest()->first();

// 获取created_at 最小的一条记录
$user = User::oldest()->first();

// 随机一条记录
$users = User::inRandomOrder()->first();
```

### 限定
```php
// 跳过前两条记录，取三条记录
$users = User::skip(2)->take(3)->get();
// 输出SQL：select * from `users` limit 3 offset 2  

// 同上
$users = User::offset(2)->limit(3)->get();
```

### 其他
```php
// 使用别名
$user = User::select("name as username", "id")->first();

// 创建一个查询构建器
$builder = User::select("name");
// 添加一个查询列到已存在的 select 子句
$user = $builder->addSelect("id")->first();
```

### 分页
```
$users = User::paginate(10);
$users = User::simplePaginate(10);
```

1. `paginate` 方法，返回`Illuminate\Pagination\LengthAwarePaginator`实例
2. `simplePaginate` 方法，返回`Illuminate\Pagination\Paginator`实例

每个分页器实例都可以通过以下方法提供更多分页信息：
```
$result->count()            // 当前页条数    
$result->currentPage()      // 当前页码
$result->perPage()          // 每页多少条
$result->total()            // 总数(使用simplePaginate 时无效)
$result->hasMorePages()     // 是否有更多
$result->firstItem()      
$result->lastItem()
$result->lastPage() (使用simplePaginate 时无效)
$result->nextPageUrl()
$result->previousPageUrl()
$result->url($page)
```

## 插入

单条插入：
```php
$user = new User();
$user->name = "yumi";
$user->fill(["email" => "yumi@example.com"]);
$user->save();
// 返回模型对象

$user = new User(
    ["name"=>"boo", 'email' => 'boo@example.com']
);
$user->save();

$result = User::create(
    ["name"=>"boo", 'email' => 'boo@example.com']
);
// 返回模型对象

// 单条插入，并返回对应 ID
$result = User::insertGetId(
    ["name"=>"boo", 'email' => 'boo@example.com']
);
// 返回插入记录对应ID
```

批量插入：
```php
$result = User::insert([
    ["name"=>"boo", 'email' => 'boo@example.com']
    ["name"=>"yumi", 'email' => 'yumi@example.com']
]);
// 返回Boolean
```
注意⚠️：此时不会触发saving、saved 模型事件

## 更新

单条更新
```php
$user = User::find(1);
$user->name = 'yumi';
$user->save();
// 返回Boolean

$user = User::find(1);
$user->update($data);
// 返回受影响行数

$user = User::where("id", 1)->update(['password' => bcrypt(122410)]);
// 返回受影响行数
```

批量更新：
```php
$user = User::whereIn("id", [1,2,3])->update(['password' => bcrypt(122410)]); 
// 返回受影响行数
```

## 删除

单个删除
```php
// 通过主键查询，删除模型
$user = User::find(1);
$user->delete();
// 返回Boolean

// 直接通过主键删除
User::destroy(1);
// 返回受影响行数

User::where('id', 1)->delete();
```

批量删除：
```php
User::destroy([1, 2, 3]);

User::destroy(1, 2, 3);
// 注：通过 Eloquent 批量删除时，deleting 和 deleted事件不会被触发，因为在进行模型删除时不会获取模型。

User::whereIn('id', [1, 2, 3])->delete();
// 均返回受影响行数
```

### 软删除
除了真实删除数据库记录，Eloquent 也可以「软删除」模型。软删除的模型并不是真的从数据库中删除了。 事实上，是在模型上设置了 `deleted_at` 属性并将其值写入数据库。如果 `deleted_at` 值非空，代表这个模型已被软删除。

如果要开启模型软删除功能，需要做好三件事情：
1. 数据库增加`deleted_at` 字段
2. 在模型上导入 `Illuminate\Database\Eloquent\SoftDeletes`特征
3. 同时将`deleted_at` 列添加到 `$dates` 属性

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use SoftDeletes;
    
    protected $dates = ['deleted_at'];
}
```

现在，当在模型实例上使用 `delete` 方法，当前日期时间会写入 `deleted_at` 字段。同时，查询出来的结果也会自动排除已被软删除的记录。

### 软删除常见操作
```php
// 验证给定的模型实例是否已被软删除
if ($user->trashed()) {
    //
}

// 包括已软删除的模型
$users = User::withTrashed()->get();
            
// 只检索软删除模型           
$users = User::onlyTrashed()->get();

// 永久删除
$user->forceDelete(); 
```

注意⚠️：
1. 通过 Eloquent 批量删除时，deleting 和 deleted 事件不会被触发，因为在进行模型删除时不会获取模型。
2. 通过 Eloquent 批量更新时，更新的模型不会触发 saving, saved, updating 和 updated 模型事件。这是因为在批量更新时实际上从未检索模型。

## 参考链接
* [Eloquent 快速入门](https://learnku.com/docs/laravel/8.x/eloquent/9406#soft-deleting)
* [Laravel 中Eloquent ORM 相关操作](https://segmentfault.com/a/1190000014916636)