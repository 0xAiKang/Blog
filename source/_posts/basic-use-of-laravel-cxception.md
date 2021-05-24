---
title: Laravel Exception 基本使用
date: 2021-05-22 20:46:20
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

在Laravel 中，所有异常都是由 `App\Exceptions\Handler` 类处理，同时也会记录在日志信息中。

<!-- more -->

通常可能会直接使用 `throw new \Exception` 来抛出一个异常终止流程，但是由于系统可能会有各式各样的异常，业务代码处处抛出 `\Exception` 和捕获 `\Exception`，导致如果遇到系统错误，无法及时通知。

异常可以大致分为两类： 用户异常 和 系统异常。

### 用户错误行为触发的异常
比如访问一个不存在的资源，对于此类异常我们需要把触发异常的原因告知用户。

可以把这类异常命名为 `InvalidRequestException`，在Laravel 中，可以通过 `make:exception` 命令来创建异常：
```
php artisan make:exception InvalidRequestException
```

`app/Exceptions/InvalidRequestException.php`：
```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;

class InvalidRequestException extends Exception
{
    public function __construct(string $message = "", int $code = 400)
    {
        parent::__construct($message, $code);
    }

    public function render(Request $request)
    {
        if ($request->expectsJson()) {
            return response()->json(['msg' => $this->message], $this->code);
        }
      
        return view('pages.error', ['msg' => $this->message]);
    }
}
```

注意这里重写了`Illuminate\Foundation\Exceptions\Handler` 父类的`render()` 方法，异常被触发时系统会调用 `render()` 方法来输出，可以在`render()` 里判断如果是 AJAX 请求则返回 JSON 格式的数据，否则就返回一个错误页面。

当异常触发时 Laravel 默认会把异常的信息和调用栈打印到日志（`storage/logs/laravel.log`）里，比如：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210522110729.png)

比如用户异常并不是因为系统本身的问题导致的，不会影响系统的运行，如果大量此类日志打印到日志文件里反而会影响我们去分析真正有问题的异常，因此需要屏蔽这个行为。

在`app/Exceptions/Handler.php`类中，将需要屏蔽的类加入到`dontReport` 属性中：
```php
protected $dontReport = [
      InvalidRequestException::class,
  ];
```

### 系统内部异常
比如连接数据库失败，或者某SQL 执行异常，对于此类异常需要有限度地告知用户发生了什么，因此，可以传入两条信息，一条是给用户看的，另一条是打印到日志中给开发人员看的。

新建一个 `InternalException` 类：
```
$ php artisan make:exception InternalException
```

`app/Exceptions/InternalException.php`：
```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;

class InternalException extends Exception
{
    protected $msgForUser;

    public function __construct(string $message, string $msgForUser = '系统内部错误', int $code = 500)
    {
        parent::__construct($message, $code);
        $this->msgForUser = $msgForUser;
    }

    public function render(Request $request)
    {
        if ($request->expectsJson()) {
            return response()->json(['msg' => $this->msgForUser], $this->code);
        }

        return view('pages.error', ['msg' => $this->msgForUser]);
    }
}
```
在该类中，只需要传入真正的异常，记录到日志中，而最终返回给用户的只有『系统内部错误』这些信息。

### 实际应用
假设在控制器中，需要调用一个封装好的API 类，在该类中，使用`\Exception` 抛出异常，那么在控制器中，可以使用我们自定义的`InternalException` 类进行接管异常。

```php
<?php
namespace App\Http\Controllers\Api;

use App\Services\Api\UserApiService;

class UsersController extends Controller
{
    // ...
    
    public function store(UserRequest $request)
    {
         try {
            // ...
            
            UserApiService::doSomething();
        } catch (\Exception $exception) {   // 接管 \Exception 异常
            // 抛出自定义异常
            // $exception->getMessage() 为 UserApiService 抛出的具体异常信息
            throw new InternalException($exception->getMessage());
        }
    }
}
```

客户端触发异常：
```json
{
    "code": 500,
    "msg": "系统内部异常"
}
```

### 参考链接
* [Laravel 优雅地处理异常](https://learnku.com/courses/laravel-shop/8.x/exceptions/10097#0c45e2)