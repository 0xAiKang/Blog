---
title: Laravel Request 常见表单验证整理
date: 2021-05-24 21:11:08
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

Laravel 的表单验证非常强大，结合单一职责原则，在Request 类中就能满足日常开发绝大多数验证场景。

<!-- more -->

下面整理了常用的一些表单验证规则，大致可以分为以下几类：
1. 常规验证
2. 自定义规则验证
3. 数据库验证

## 常规验证
所谓的常规验证就是直接使用Laravel 表单验证为我们提供的验证规则。

### boolean
验证的字段必须可以转换为 Boolean 类型。 可接受的输入为 `true` ， `false` ， `1` ，`0` ， `"1"` 和 `"0"` 。

### alpha
待验证字段只能由字母组成。

### alpha_num
待验证字段只能由字母和数字组成。

### array
待验证字段必须是有效的 PHP 数组。

### url
验证的字段必须是有效的 URL。

### integer
验证的字段必须是整数。

### numeric
验证字段必须为数值。

注意数值和整数的区别。

### json
验证的字段必须是有效的 JSON 字符串。

### max/min
验证字段必须在最小值与最大值之间。

```php
return [
    "value" => "max:99|min:1"    
];
```

### in
验证字段必须包含在给定的值列表中：

```php
return [
    "value" => "in:0,1,2,3"    
];
```

### digits_between
验证中的字段必须为 numeric，并且长度必须在给定的 min 和 max 之间。

```php
return [
    "mobile" => "digits_between:8,11"    
];
```

### dimensions
验证的文件必须是图片并且图片比例必须符合规则:

```php
return [
    'avatar' => 'dimensions:min_width=100,min_height=200'
];
```
对于图片的验证，使用 `dimensions` 尤其有用。

### image
验证的文件必须是图片 (jpeg, png, bmp, gif, svg, or webp)。

### required
验证的字段必须存在于输入数据中，而不是空。如果满足以下条件之一，则字段被视为「空」：

* 值为 null。
* 值为空字符串。
* 值为空数组或空 Countable 对象。
* 值为无路径的上传文件。

### filled
验证的字段在存在时不能为空。

### exclude_if
可以指定当验证字段的值为 value 时，其他验证规则可以排除。

```php
return [
    "id" => "required|exclude_if:id,0|integer|exists:admin,id",
];
```
当 `id` 为零时，不会验证`exists` 规则。

## 自定义规则验证
如果某个规则仅仅只使用一次，那么使用闭包来创建自定义规则再适合不过，闭包函数接收属性的方法，属性的值以及在校验失败时的回调函数 `$fail`：

```php
return [
    "age" => function ($attribute, $value, $fail) {
          if ($value > 18) {
              return $fail("年龄不符");
          } 
     };
];
```

## 数据库验证

### exists
验证的字段必须存在于给定的数据库表中。

```php
return [
    'email' => 'exists:users,email_address'
];
```

### unique
验证字段在给定的数据库表中必须是唯一的。

语法：`unique:table,column,except,idColumn`。

基本用法，指定自定义的列表：
```php
return [
    'email' => 'unique:users,email_address'
];
```

在做`update` 操作时，如果提交了 `email`，那么上面的那个验证仍然会生效，这时可以通过定义  `except` 当前用户。

```php
return [
    'email' => 'unique:users,email_address,$this->uid,uid'
];
```

生成的SQL：
```php
select count(*) as aggregate from users where email_address = "geeek001@qq.com" and uid <> 1;
```

同样可以使用`Rule` 助手函数来完成数据库验证：

```php
return [
    "email" => [
          Rule::unique("users", "email_address"),
      ],    
];
```

## 参考链接
* [Laravel 表单验证](https://learnku.com/docs/laravel/8.x/validation/9374#6cc7dc)