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

## 问题五
Laravel 好像自从`7.x` 版本开始，为了格式化日期以进行序列化，框架使用了新的日期序列化格式——Carbon 的 `toJSON` 方法。

使用新格式序列化的日期将显示为：`2021-07-11 20:01:002019-12-02T20:01:00.283041Z`，眼熟不，这个格式。

如果还想使用正常的`年-月-日 时:分:秒` 格式，则可以在模型上覆盖 `serializeDate` 方法即可：
```php
use DateTimeInterface;

/**
 * Prepare a date for array / JSON serialization.
 *
 * @param  \DateTimeInterface  $date
 * @return string
 */
protected function serializeDate(DateTimeInterface $date)
{
    return $date->format('Y-m-d H:i:s');
}
```

## 问题六

在Laravel Model 中，将某个属性设置为`array casting`：
```php
protected $casts = [
    'options' => 'array',
];
```

这时如果再想对其值进行修改，就会引发异常：
```php
$data->options["key"] = "value";

// production.ERROR: Indirect modification of overloaded property
```

可见，`casting` 并不支持一些针对特定类型的操作，例如无法作为指定类型的函数的参数。

按照官方文档的做法，应该是先赋值给一个中间变量，进行操作，然后再赋值回去。

```php
$user = App\User::find(1);
$options = $user->options;
$options['key'] = 'value';
$user->options = $options;
$user->save();
```

## 问题七

`queue:listen` 与 `queue:work` 有什么区别？

这是一个关于队列的问题，前者与后者的区别在于，当上下文环境发生变化，前者会自动重新加载新的上下文，而后者则不会。

自Laravel `5.x` 版本以来，官方文档中已不再介绍`queue:listen` 指令怎么使用了，所以开发阶段建议使用 `queue:listen` 进行调试，其余情况建议全部使用 `queue:work`，因为效率更高。

## 问题八
Laravel 如何在关联模型中排序？

答案是：对于跨表排序这种需求，模型关联默认是没有实现的，因为模型关联的原理是将SQL 拆分成两条，模型关联的结果集是基于前面一条SQL 返回的id 集。

通常有两种方式解决以上需求：
1. 冗余字段
2. 使用 join

这里顺带介绍一个 Builder marco，也可以解决以上问题：
```php
// 基于关联关系排序实现
Builder::macro('orderByWith', 
    function ($relation, $column, $direction = 'asc'): Builder{
            /** @var Builder $this */
            if (is_string($relation)) {
                $relation = $this->getRelationWithoutConstraints($relation);
            }

            return $this->orderBy(
                $relation->getRelationExistenceQuery(
                    $relation->getRelated()->newQueryWithoutRelationships(),
                    $this,
                    $column
                ),
                $direction
            );
        });
```

调用方式如下：
```php
// 基于当前分类关联
$products = Product::has('cates')
    // 根据分类关联中的sort字段进行排序
    ->orderByWith('cates', 'sort', 'desc')
    ->paginate($limit);
```

## 问题九
Observer 还是 Listener？

Observer 可以监听 Eloquent 模型的 `creating`、`created`、`saving`、`saved` 等事件，而这些Listener 其实也可以做到，那么对于两者该如何选择呢？

其实通过观察Observer的逻辑就会发现，它只是在帮助你添加了 listen 的逻辑，帮助你简化了事件 Listener 的注册。

所以我通常是这么选择的：如果对应场景是 Eloquent 模型的相关事件，则会直接选择 Observer; 如果对应场景是业务事件触发，则会选择Listener。

## 参考链接
* [Laravel attribute casting 导致的 Indirect modification of overloaded property](https://www.cnblogs.com/sgm4231/p/10194746.html)
* [Laravel 怎么通过关联字段排序？](https://learnku.com/laravel/t/9290/how-can-laravel-be-sorted-by-associated-fields)