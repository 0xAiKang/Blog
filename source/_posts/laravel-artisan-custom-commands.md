---
title: Laravel Artisan 自定义命令
date: 2021-06-05 09:39:06
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

Laravel 主要提供了三种实现命令行交互的工具：
1. Artisan：内置一套命令行操作，并可以自定义进行扩展
2. Tinker：为应用提供了 REPL 或者交互的 shell
3. 安装器：通常在项目初始化时用到

本章的内容主要以 Artisan 命令为主。

<!-- more -->

## Artisan 基本命令
* `help` 帮助命令，例如 `php artisan help commandName`
* `clear-compiled` 删除 Laravel 的编译文件（就像一个内部缓存），当遇到一些奇怪的问题时，可以先尝试运行这个命令
* `down` 把应用切换到『维护模式』以解决错误、迁移或者其他运行方式。up 可以在『维护模式』里恢复应用
* `env` 显示当时Laravel 的运行环境，它等效于在应用中输入 `app()->environment()`
* `migrate` 迁移数据库
* `optimize` 通过把重要的PHP 类缓存到 `bootstrap/cache/compile.php` 来优化应用
* `serve` 部署一个PHP 服务器到 `localhost:8000`（可以通过 --host 和 -port 自定义修改主机名和端口号）
* `tinker` 打开Tinker 的REPL

## 组合命令
Laravel 内置了许多好用的组合命令。

### config
为了更快地查阅，`config:cache` 会缓存所有的配置，`config:clear` 会清理缓存。

### db
如果已经配置了数据库的 `seeder`，便可以用 `db:seed` 命令，来填充数据库。

### key
`key:generate` 会在 `.env` 文件中创建一个随机的应用加密密钥，用于对数据进行加密。

> 注意：这个命令只需要运行一次，也就是初始环境时，如果再次运行，则会丢失原有的密钥。

### make

* `make:command`：创建一个新的 Artisan 命令
* `make:controller`：生成 Controller
* `make:request`：生成 Request
* `make:resource`：生成 Resource
* `make:exception`：生成 Exception
* `make:job`：创建延迟任务

### queue
* `queue:listen`：开始监听一个队列
* `queue:table`：为数据库支持队列创建一个迁移
* `queque:flush`：刷新所有失败的队列任务

### route
* `route:list`：查看应用的每一个路由定义，包括每个路由器的方法、路径、名字、控制器/闭包动作和中间件。
* `route:cache`：缓存路由器的定义，以便更快地查阅
* `route:clear`：清理控制器


## 自定义Artisan 命令

先来创建一个新的命令：
```shell
php artisan make:command yourCommand
```

先来看一下Artisan 命令的默认架构：
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class yourCommand extends Command
{
    /**
     * 控制台命令名称和签名
     *
     * @var string
     */
    protected $signature = 'command:name';

    /**
     * 控制台命令描述
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * 创建一个新的命令实例
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * 执行控制台命令
     *
     * @return int
     */
    public function handle()
    {
        return 0;
    }
}
```

* `$signature`：用于定义命令签名，比如签名是 `command:name`，那么最终执行的命令应该是：`php artisan command:name`
* `$description`：对于该命令的描述
* `handle()`：执行命令所需要做的事情

在 `handle()` 中检索命令行参数和选项值：
* `argument()`：返回一个包含所有参数的数组
* `option()`：返回一个包含所有选项的数组

在 `handle()` 中获取用户输入：
* `ask()`：提示用户输入文本
* `secret()`：提示用户输入文本，但是会用星号来隐藏输入内容
* `confirm()`：提示用户恢复 是/否，返回一个布尔值
* `choice()`：提示用户选择一个选项，如果用户没有选择，那么最后一个参数就会使用默认值

### Code generator 
Laravel 默认为我们提供了，`make:controller` 这样的生成控制器的命令，那如果需要生成自定义的代码，又该如何做呢？

这里以 `Service` 为例，首先创建命令：
```shell
php artisan make:command MakeService
```

编辑`app/Console/Command/MakeService.php`：
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\GeneratorCommand;

/**
 * Generator Service
 *
 * Class MakeService
 *
 * @package App\Console\Commands
 */
class MakeService extends GeneratorCommand
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $name = 'make:service';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Create a new custom service class';

    /**
     * The type of class being generated.
     *
     * @var string
     */
    protected $type = 'Service';

    /**
     * Get the stub file for the generator.
     *
     * @return string
     */
    protected function getStub()
    {
        return __DIR__ . '/stubs/service.stub';
    }

    /**
     * Get the default namespace for the class.
     *
     * @param  string $rootNamespace
     * @return string
     */
    protected function getDefaultNamespace($rootNamespace)
    {
        return $rootNamespace . '\Services';
    }
}
```

通过实现 `GeneratorCommand` 抽象类来生成代码，同时需要在 `MakeService.php` 同级目录下创建一个 `stubs/service.stub` 文件，并填入相应的模板文件：

```
<?php

namespace {{ namespace }};

class {{ class }}
{
    protected $model;

    public function __construct()
    {
        // $this->$model = $model;
    }
}
```

最红使用 `php artisan make:service TestService` 命令，就可以生成Service 了～