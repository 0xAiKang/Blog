---
title: Laravel 常用扩展包整理
date: 2021-06-19 20:51:35
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

Laravel 开发如此高效，除了其框架本身易用之外，同时也离不开各种丰富的扩展包的支持。

<!-- more -->

## 辅助类
### laravel-cors
所有问题，跨域先行。跨域问题没有解决，一切处理都是纸老虎。

[laravel-cors](https://github.com/fruitcake/laravel-cors) 是一个解决跨域问题的扩展包，不知道是从哪个版本起，已经默认引入框架了。

```sh
composer require fruitcake/laravel-cors
```

### Laravel-lang
[Laravel-lang](https://github.com/overtrue/laravel-lang) 是一个非常易用的语言包，现已支持多达75 种语言。

```sh
composer require "overtrue/laravel-lang:~5.0"
```

编辑配置文件：`config/app.php`：
```php
'locale' => 'zh_CN',
```

### Captcha for Laravel
[captcha](https://github.com/mewebstudio/captcha) 是一个生成验证码的扩展包。

```sh
composer require mews/captcha
```

### Carbon
[Carbon](http://carbon.nesbot.com/) 可以帮助我们在 PHP 开发中处理日期 / 时间变得更加简单、更具语义化。

```sh
composer require nesbot/carbon
```

记得设置时区 `config/app.php`：
```php
'timezone' => 'PRC',
```

### Eloquent Model Generator
[Eloquent Model Generator](https://github.com/krlove/eloquent-model-generator) 是一个基于代码生成器的 Eloquent Model 生成工具。

只在开发环境中安装：
```sh
composer require --dev krlove/eloquent-model-generator
```

使用：
```sh
php artisan krlove:generate:model UserModel --table-name=user --output-path=./Models --namespace=App\\Models
```

### IDE Helper
[Laravel IDE Helper](https://github.com/barryvdh/laravel-ide-helper#laravel-ide-helper-generator) 是一个代码提示及补全工具。

只在开发环境中安装：
```sh
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

基础用法：
* `php artisan ide-helper:generate`：为 Facades 生成注释
* `php artisan ide-helper:models`：
* `php artisan ide-helper:meta`：生成 PhpStorm Meta file

#### 为 Facades 生成注释
运行一下命令：
```sh
php artisan ide-helper:generate
```

执行完成之后会在项目根目录下生成一个 `_ide_helper.php` 文件。

#### 为模型生成注解
使用Laravel 为我们提供的 `make:model` 默认不会为在模型文件中，生成相应的注解。

当我们需要通过对象获取模型的某个属性时，IDE 这时会提示未定义的属性，虽然不会影响功能的使用，但是对于开发人员来说并不友好。

看起来就像这样：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210605110602.png)

使用`IDE Helper` 来生成模型注解：
```sh
php artisan ide-helper:models "App\Models\UserModel"

Do you want to overwrite the existing model files? Choose no to write to _ide_helper_models.php instead? (Yes/No):  (yes/no) [no]:
```

建议选择『yes』，否则会生成「_ide_helper_models.php」文件，这样在跟踪文件的时候不会跳转到「_ide_helper_models.php」文件。

如果希望为所有模型都加上注解，则省略后面的参数。

看起来好多了：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210605111205.png)

> 注意： 为模型生成字段信息必须在数据库中存在相应的数据表。

#### 自动为链式操作注释

```sh
php artisan ide-helper:meta
```

执行完成之后会在项目根目录下生成一个 ` .phpStorm.meta.php` 文件。

### Laravel Query Logger

```sh
composer require overtrue/laravel-query-logger --dev
```

启用日志记录 `config/logging.php`：
```php
// 加入以下配置，开启日志查询记录
'query' => [
    'enabled' => env('LOG_QUERY', env('APP_ENV') === 'local'),

    // Only record queries that are slower than the following time
    // Unit: milliseconds
    'slower_than' => 0,

    // Only record queries when the QUERY_LOG_TRIGGER is set in the environment,
    // or when the trigger HEADER, GET, POST, or COOKIE variable is set.
    'trigger' => env('QUERY_LOG_TRIGGER'),
],
```

使用：
```sh
tail -f ./storage/logs/laravel.log
```

### Logging for PHP
[Logging for PHP](https://github.com/Seldaek/monolog) 是一个可以将日志保存至各种位置的扩展包。

通常在测试回调时，会用得比较多。
```bash
composer require monolog/monolog
```

基础用法：
```php
<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

// create a log channel
$log = new Logger('name');
$log->pushHandler(new StreamHandler('path/to/your.log', Logger::WARNING));

// add records to the log
$log->warning('Foo');
$log->error('Bar');
```

### Laravel Debugbar
Laravel Debugbar 是一个很棒的扩展包。在很多应用程序方面，你可以使用它来收集数据。比如查询，视图，时间等等；

```php
composer require barryvdh/laravel-debugbar --dev
```

### Laravel Telescope
Laravel Telescope 是一个非常酷的工具，对 Laravel 应用，有“优雅的调试助手”的美称。

你可以用它来监控很多东西：
* 请求
* 命令
* 异常
* 日志
* 查询
* 事件
* ...

```php
composer require laravel/telescope --dev
```

### Faker
[fzaninotto/Faker](https://github.com/fzaninotto/Faker) 是一个生成假数据的 PHP 库，支持非常多的语言。

```bash
composer require fzaninotto/faker
```

[Poppy Faker](https://github.com/imvkmark/poppy-faker) 是基于 [fzaninotto/Faker](https://github.com/fzaninotto/Faker) 的中文轻量级 Fake 数据生成类。

```bash
composer require poppy/faker
```

## 开发类

### jwt-auth
[jwt-auth](https://github.com/tymondesigns/jwt-auth) 是 Laravel 和 lumen 下一个优秀 JWT 组件。

具体使用介绍可以查看我的——[Laravel jwt-auth 使用详解](https://www.0x2beace.com/laravel-jwt-auth-use-detailed-explanation/) 。

### laravel-enum
[laravel-enum](https://github.com/BenSampo/laravel-enum) 是一个简单易用，扩展性高的处理枚举的扩展包。

```sh
composer require bensampo/laravel-enum
```

### Intervention/image
[Intervention/image](https://github.com/Intervention/image) 是一个处理图片裁切的扩展包，对应的API 文档在[这里](http://image.intervention.io/api/crop)。

```shell
composer require intervention/image
```

### laravel-wechat

[微信 SDK for Laravel](https://github.com/overtrue/laravel-wechat)。

```sh
composer require "overtrue/laravel-wechat"
```

### easy-sms
[easy-sms](https://github.com/overtrue/easy-sms) 一款满足你的多种发送需求的短信发送组件。

```sh
composer require "overtrue/easy-sms"
```