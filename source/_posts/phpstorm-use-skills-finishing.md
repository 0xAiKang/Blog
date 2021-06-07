---
title: PHPStorm 使用技巧整理
date: 2021-06-07 23:28:42
tags: ["PHP", "PHPStorm"]
categories: ["PHP", "PHPStorm"]
---

这篇笔记用来整理 PHPStorm 的一些使用技巧。

<!-- more -->

## 类型提示
在使用IDE 开发的过程中，不知你是否有注意到这样一个问题：

```php
public function article(){
    return $this->hasMany(User::class)->where(function($query){
        $query->where('open',1);
    });
}
```

在使用`$query` 调用 `where` 方法时，默认是没有提示的，这是为什么呢？

这是因为PHP 语言特性的原因，一个数组可以存放各种类型的值，无法从外部知道里面的值具体是什么类型，这就导致IDE 无法给出有效的提示了。

其实这时我们只需要显示的告诉IDE，这个变量具体是什么类型的，编译器就能正常提示了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210605113822.png)

```php
/** @var Collection $collection */
```

## IDE Helper
[Laravel IDE Helper](https://github.com/barryvdh/laravel-ide-helper#laravel-ide-helper-generator) 是一个代码提示及补全工具。

只在开发环境中安装：
```shell
composer require --dev barryvdh/laravel-ide-helper
```

对于只在开发环境中需要安装的扩展包，在 app/Providers/AppServiceProvider.php 文件中以如下方式进行注册：
```php
public function register()
{
    if ($this->app->environment() !== 'production') {
        $this->app->register(\Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider::class);
    }
}
```

### 为 Facades 生成注释
运行一下命令：
```shell
php artisan ide-helper:generate
```

执行完成之后会在项目根目录下生成一个 `_ide_helper.php` 文件。

### 为模型生成注解
使用Laravel 为我们提供的 `make:model` 默认不会为在模型文件中，生成相应的注解。

当我们需要通过对象获取模型的某个属性时，IDE 这时会提示未定义的属性，虽然不会影响功能的使用，但是对于开发人员来说并不友好。

看起来就像这样：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210605110602.png)

使用`IDE Helper` 来生成模型注解：
```shell
php artisan ide-helper:models "App\Models\UserModel"

Do you want to overwrite the existing model files? Choose no to write to _ide_helper_models.php instead? (Yes/No):  (yes/no) [no]:
```

建议选择『yes』，否则会生成「_ide_helper_models.php」文件，这样在跟踪文件的时候不会跳转到「_ide_helper_models.php」文件。

如果希望为所有模型都加上注解，则省略后面的参数。

看起来好多了：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210605111205.png)

> 注意： 为模型生成字段信息必须在数据库中存在相应的数据表。

### 自动为链式操作注释

```php
php artisan ide-helper:meta
```

执行完成之后会在项目根目录下生成一个 ` .phpStorm.meta.php` 文件。