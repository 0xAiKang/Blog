---
title: Laravel 连接多个 Mysql 数据库
date: 2021-08-18 08:18:42
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

在Laravel 项目里面，如果需要使用多个数据库，其实是很简单的事情。

<!-- more -->

第一步，首先需要在 `.env` 文件中定义第二个数据库连接：
```php
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=database_name
DB_USERNAME=root
DB_PASSWORD=

DB_HOST_CENTER=127.0.0.1
DB_PORT_CENTER=3306
DB_DATABASE_CENTER=database_center
DB_USERNAME_CENTER=root
DB_PASSWORD_CENTER=
```

第二步，在 `config/database.php` 文件中配置第二个数据库连接：
```php
'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', 'localhost'),
        'port' => env('DB_PORT', '3306'),
        'database' => env('DB_DATABASE', 'forge'),
        'username' => env('DB_USERNAME', 'forge'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix' => '',
        'strict' => false,
        'engine' => null,
],

'mysql_center' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST_CENTER', 'localhost'),
        'port' => env('DB_PORT_CENTER', '3306'),
        'database' => env('DB_DATABASE_CENTER', 'forge'),
        'username' => env('DB_USERNAME_CENTER', 'forge'),
        'password' => env('DB_PASSWORD_CENTER', ''),
        'charset' => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix' => '',
        'strict' => false,
        'engine' => null,
],
```
### 使用方式一

在模型中指定需要连接的数据库：
```php
// 如果没有指定数据库连接，则会使用默认的 mysql 连接
class UserModel extends Model
{
      // 数据库'database'中的users表
      protected $table = "users";
}

// 指定mysql_center 作为数据库连接
class UserModel extends Model
{
      // 数据库'dadtabase_center'中的users表
      protected $connection = 'mysql_center';
      
      protected $table = "users";
}
```

这样，当再次使用该模型时，其背后连接的就是新的数据库。

```php
UserModel::find(1);

// [2021-08-17 21:32:53] DEV.DEBUG: [database_center] [54.96ms] select * from `user` where `uid` = '1' 
```
### 使用方式二

上面是为模型指定数据库连接，那么有没有什么办法为单个查询指定连接？

也是可以的。

使用该方式之前，也需要提前配置好需要连接的数据库
```php
$data = DB::connection("mysql_center")
    ->table("user")
    ->get();
```

这样就可以在不改变模型属性的情况下，查询到其他数据库的数据了。

## 参考链接
* [Laravel 数据库：连接多个 MySQL 数据库](https://learnku.com/laravel/wikis/16106)