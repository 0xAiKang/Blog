---
title: Laravel 完善 Error/Exception 的捕获与处理
date: 2021-08-29 16:19:29
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

前端时间，线上的管理后台的某个功能突然被告知无法正常使用。

<!-- more -->

抛出的异常如下：
> Trying to access array offset on value of type null

初步判定就是因为某个数组或者对象的值为`null`，所导致。

于是马上打开`laravel.log` 系统日志文件，想从这里站牌找到异常原因。

找遍整个文件都没有发现相关字眼，这时我才意识到，本来Laravel 对于运行时错误，是会进行记录的，但是我将异常捕获到之后，并没有写入日志文件，而是直接返回给客户端了...

显然这对于后面排查问题相当不方便，所以必须做点什么，将异常信息给保存起来。

---------

PHP 系统级用户代码的错误类型有两种，可由 `try ... catch ...` 进行捕获。
* `E_PARSE`：解析时错误（语法解析错误）通常因为少个分号、多个逗号之类的问题导致的致命错误
* `E_ERROR`：运行时错误，通常是因为调用了未定义的函数或方法而引发的致命错误。

像文章开头的那个异常，就属于：**运行时错误**。这类异常正是我们需要捕获并记录的。

因为目前的代码已经实现了，捕获系统异常并返回给客户端等相关功能，所以剩下需要完成的就只是写入日志文件。

由于并不是所有异常都需要记录，比如由用户行为而造成的异常，就无需额外记录，所以最终实现如下：
```php
<?php

namespace App\Exceptions;

use App\Traits\ApiResponse;
use Exception;

/**
 * 系统内部异常
 *
 * Class InternalException
 *
 * @package App\Exceptions
 */
class InternalException extends Exception
{
    private array $doReport = [
        \Exception::class,       // 所有异常的基类
        \ErrorException::class,  // 错误异常
    ];

    /**
     * InternalException constructor.
     *
     * @param string $message
     * @param int    $code
     */
    public function __construct($message = "系统内部错误", $exception = null, $code = 500)
    {
        if ($exception) {
            if (in_array(get_class($exception), $this->doReport)) {
                \Log::error("系统内部异常", ["exception" => $exception]);
            }
        }

        parent::__construct($message, $code);
    }
}
```

调用如下：
```php
public function say()
{
    try {
        // do something
    } catch (\Exception $exception) {
        throw new InternalException($exception->getMessage(), $exception);
    }
}
```

这样，如果后面如果再次遇到类似的问题，就可以直接去`laravel.log` 文件中找到具体的异常信息了。