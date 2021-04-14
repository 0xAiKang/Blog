---
title: Laravel Eloquent 常用属性整理
date: 2021-04-14 23:11:13
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

Eloquent 提供了很多属性，通过对模型进行约定，可以实现很多很方便的功能。

<!-- more -->

### connection
```php
 /**
  * 为模型指定一个连接名称
  *
  * @var string
  */
 protected $connection = 'connection-name';
```

### table
```php
/**
 * 为模型指定一个表名
 *
 * @var string
 */
 protected $table = 'users';
```

### primaryKey
```php 
/**
 * 为模型指定主键
 *
 * @var string
 */
 protected $primaryKey = 'user_id';
```

### incrementing
Eloquent 假设主键是一个自增的整数值，这意味着默认情况下主键会自动转换为 int 类型。

如果希望使用非递增或非数字的主键则需要设置公共的 `$incrementing` 属性设置为 false：
```php
 /**
  * 如果使用的是非递增或者非数字的主键
  *
  * @var bool
  */
 public $incrementing = false;
```

### keyType
如果你的主键不是一个整数，你需要将模型上受保护的 `$keyType` 属性设置为 string：
```php
 /**
  * 自定义主键类型
  *
  * @var string
  */
 protected $keyType = 'string';
```

### timestamps
默认情况下，Eloquent 预期你的数据表中存在`created_at` 和`updated_at` 字段，如果不想让 Eloquent 自动管理这两个列， 请将模型中的 $timestamps 属性设置为 false：
```php
/**
 * 是否主动维护时间戳
 *
 * @var bool
 */
public $timestamps = false;
```

### CREATED_AT|UPDATED_AT
```php
// 自定义存储时间戳的字段名
const CREATED_AT = 'start_time';
const UPDATED_AT = 'end_time';
```

### dateFormat
如果需要自定义时间戳的格式，在你的模型中设置 $dateFormat 属性。这个属性决定日期属性在数据库的存储方式，以及模型序列化为数组或者 JSON 的格式：
```php
/**
 * 模型日期的存储格式
 *
 * @var string
 */
protected $dateFormat = 'U';
```
不清楚 U 是什么意思的，请看 [Date/Time 函数](http://php.net/manual/zh/function.date.php) 。

### attributes
```php
/**
 * 模型属性的默认值
 *
 * @var array
 */
protected $attributes = [
    'delayed' => false,
];
```

### hidden
```php
 /**
  * 隐藏以下字段
  *
  * @var array
  */
 protected $hidden = ['password'];
```

### visible
```php
 /**
  * 显示以下字段
  *
  * @var array
  */
 protected $visible = ['first_name', 'last_name'];
```
如果说`$hidden` 属性是黑名单，那么`$visible` 就是白名单。

### fillable

```php
/**
 * 可以被批量赋值的属性
 *
 * @var string[]
 */
protected $fillable = ["username"];
```

### guarded

```php
 /**
  * 设定不可被批量赋值的属性，当 $guarded 为空数组时则所有属性都可以被批量赋值。
  *
  * @var array
  */
 protected $guarded = ['price'];    
```

### casts
```php
 /**
  * 字段转换为对应的类型
  *
  * @var array
  */
 protected $casts = [
    "settings" => "array",
    'created_at' => 'datetime:Y-m-d H:i:s',
    'updated_at' => 'datetime:Y-m-d H:i:s',
    'is_admin' => 'boolean',
 ];
```

### dates
```php
 /**
  * 需要转换成日期的属性。
  *
  * @var array
  */
 protected $dates = ['deleted_at'];
```

### perPage
```
 /**
  * 默认分页数量
  *
  * @var int
  */
 protected $perPage = 50;
```

### touches
```php
 /**
  * 更新关联模型的 updated_at 字段
  *
  * @var array
  */
 protected $touches = ['post'];
```

### dispatchesEvents
```php
 /**
  * 模型的事件映射
  *
  * @var array
  */
 protected $dispatchesEvents = [
     'saved' => UserSaved::class,
     'deleted' => UserDeleted::class,
 ];
```

### with

为关联模型默认添加『渴求式加载』，等效于使用查询构造器时，手动指定`with`。
```php
class User {
  /**
   * 
   *   
   * @var string[] 
   */
  protected $with = [
      "topics",
  ];

  public function topics()
  {
      return $this->hasMany(Topic::class);
  }
}
```

### 参考链接
* [Eloquent ORM 快速入门](https://learnku.com/docs/laravel/8.x/eloquent/9406#e45381)