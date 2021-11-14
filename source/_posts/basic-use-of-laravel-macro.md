---
title: Laravel Macro 基本使用
date: 2021-11-14 08:35:20
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

在实际开发中，如果想扩展一个基础类，通常的做法是先继承该类，然后在此基础上进行扩展。

而本文要介绍的宏指令，是Laravel 根据PHP 的特性，编写了一套叫做 `Macroable` 的 `Traits`，可以在不使用继承的情况下进行扩展。

<!-- more -->

## 如何使用

定义一个宏指令：
```php
<?php
use Illuminate\Http\Request;

Request::macro("sayHi", function ($name) {
    echo "Hi " . $name . "!";
});
```

使用它：
```php
<?php
<?php
use Illuminate\Http\Request;

Request::sayHi("Boo");   // Hi Boo!
```

上面那个例子非常简单，再来看一个复杂一些的，将集合的数组全部转换为大写：
```php
<?php
use Illuminate\Support\Collection;

Collection::macro('uppercase', function () {
    // $this->items 就相当于是集合的所有元素 ["hello", "world"]
    return collect($this->items)->map(function ($item) {
        return strtoupper($item);
    });
});

collect(["hello", "world"])->uppercase();   // ["HELLO", "WORLD"]
```

### 关于Macro 的$this
这里有必要说一下`$this` 的指向，在上面那个集合的示例中，`$this` 并不是指向当前类，而是指向当前集合。

这是因为在 `Marcoable` 的源代码中，是可以看到 `static::$macros[$method]->bindTo($this, static::class)` 这段代码。而 `bindTo` 是改变 `$this` 上下文指向的方法。

## 应该放在哪里
有两个地方可以用来存放有关宏指令的代码，分别是：

### 利用composer 自动加载
第一种是使用通过 Composer autoload 的简单 PHP 文件。

可以在app 目录下新建一个叫做`macros.php` 的文件，然后编辑 `composer.json` 添加一个 `files`属性（注意相对路径 `app/macros.php`）：
```json
"autoload": {
    "psr-4": {
        "App\\": "app/"
    },
    "classmap": [
        "database/seeds",
        "database/factories"
    ],
    "files": [
        "app/macros.php"
    ]
}
```
最后运行 `composer dump-autoloader`。

这样新添加的文件将在运行时加载和执行，整个应用都可以使用其中的宏指令。

### 利用服务提供者
创建一个 `ServiceProvider`，并注册在`config/app.php` 中，

比如创建一个集合的服务提供者，加入`boot` 方法
```php
<?php
namespace App\Providers;

use Collection;
use Illuminate\Support\ServiceProvider;

class CollectionMacroServiceProvider extends ServiceProvider {

    public function boot()
    {
       Collection::macro('uppercase', function () {
          return collect($this->items)->map(function ($item) {
              return strtoupper($item);
          });
      });
    }
}
```

这样，整个应用就都可以访问到了。

## 有哪些类可以使用Macro

任何使用 `Macroable` Laravel 框架中的`Traits` 的类都可以使用宏，比如：
* Collection
* Builder
* Response
* Rule
* Request
* Route
* HTML
* Form
* Filesystem
* Cache
* Str
* Arr
* Translator
* ...

Macro 是Laravel 中又一强大而被忽视的存在，合理地使用Macro 以实现更好的表达和复用。

## 参考连接
* [如何利用 macro 方法来扩展 Laravel 的基础类的功能](https://learnku.com/laravel/t/2915/how-to-use-the-macro-method-to-extend-the-function-of-the-base-class-of-laravel)
* [Laravel Tip: 5 examples of why you should be using Macros](https://medium.com/@SlyFireFox/laravel-tip-5-examples-of-why-you-should-be-using-macros-90e015d1bce)
