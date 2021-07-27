---
title: Laravel Eloquent 高阶用法整理
date: 2021-07-27 08:31:56
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

这篇笔记用来整理Laravel Eloquent 的一些高级用法。

<!-- more -->

## hidden/visible
有时可能会遇到需要隐藏/显示属性的需求，尽管Eloquent ORM 为我们提供了`hidden`、`visible` 属性，但如果能动态设置，似乎更不错。

`src/Illuminate/Database/Eloquent/Concerns/HidesAttributes.php` Tarit 为我们提供了几个不错的方法：
* `getVisible`：获取白名单
* `setVisible`：设置白名单
* `makeVisible`：追加白名单
* `getHidden`：获取黑名单
* `setHidden`：设置黑名单
* `makeHidden`：追加黑名单

### 最佳实践

```php
$user = UserModel::find(1)->makeHidden(["remember_token"]);

$user->remember_token;
// obvQx5ZVZAoAVmWwkB-STy8xVPV1
```

> 需要注意一点的是，虽然属性被我们隐藏了，但如果仍需要使用该属性的话，还是可以通过`->` 获取到

## Accessors
要定一个Accessors，需要在模型中创建一个名称为`getXxxAttribute` 的方法，其中的 `Xxx` 是驼峰命名法的字段名。

通过属性获取器，我们可以很轻松地为属性赋予新的值，但如果想要获取赋值之前的值，那么该如何做呢？

```php
/**
 * @return string
 */
public function getContentAttribute($value)
{
    return strip_tags($value);
}
```

`src/Illuminate/Database/Eloquent/Concerns/HasAttributes.php` Tarit 为我们提供了几个不错的方法：
* `getAttributes`：获取赋值之前的值
* `getAttribute`：获取指定Key，修饰之后的值
* `getAttributeValue`：获取指定Key，修饰之后的值
* `setAttribute`：为属性赋值
* `getMutatedAttributes`：获取需要赋值的Key

> 需要注意的是：以上这些Api 仅适用于模型的实例对象，对于集合不能直接使用

```php
UserModel::find(1)->getAttributes();     ✅

UserModel::get()->getAttributes();       ❎
```

### 最佳实践

```php
$notice = NoticeModel::find(1);

$notice->content;     // Hello World

$notice->getAttribute("content");   // <p>Hello World</p>
```

## 分页
数据分页有多种方法，最简单的是使用 [查询构造器](https://learnku.com/docs/laravel/laravel/8.x/queries) 或 [Eloquent query](https://learnku.com/docs/laravel/laravel/8.x/eloquent) 的 `paginate` 方法。

但有些时候，因为一些原因，我们不想使用 `paginate` 自动创建分页，那有没有什么办法可以手动创建分页呢？

答案是有的，Laravel 提供以下两种方式：
* `Illuminate\Pagination\Paginator`：相当于查询构造器或 Eloquent 的 `simplePaginate` 方法。
* `Illuminate\Pagination\LengthAwarePaginator`：相当于查询构造器或 Eloquent 的 `paginate` 方法。

### 最佳实践

```php
UserModel::where("age", ">", 18)->paginate(15);

// 等效于
$data = UserModel::where("age", ">", 18)->get();
new LengthAwarePaginator($data, $data->count(), 15);

// 等效于
$data = UserModel::where("age", ">", 18)->get();
new Paginator($data, 15);
```