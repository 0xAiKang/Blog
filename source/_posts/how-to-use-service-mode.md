---
title: 『转载』如何使用Service 模式
date: 2021-04-12 23:22:55
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

若将数据库逻辑都写在 `Controller` 里，会造成 `Controller` 代码的臃肿难以维护，基于 `SOLID` 原则，我们应该使用 `Service` 模式辅助 `Controller`，将相关的业务逻辑封装在不同的 `Service`，方便项目的后期维护。

<!-- more -->

## 商业逻辑
商业逻辑中，常见的如 :
1. 牵涉到外部行为 : 如发送Email，使用外部API...
2. 使用PHP写的逻辑 : 如根据购买的件数，有不同的折扣。

若将商业逻辑写在 controller，会造成 controller 肥大，日后难以维护。

## Service

### 牵涉到外部的行为
如 `发送Email`，常常会在 `Controller` 中直接调用 `Mail::queue()`：
```php
 /**
 * @param \Illuminate\Http\Request $request
 */
public function store(Request $request)
{
    \Mail::queue('email.index', $request->all(), function (Message $message) {
        $message->sender(env('MAIL_USERNAME'));
        $message->subject(env('MAIL_SUBJECT'));
        $message->to(env('MAIL_TO_ADDR'));
    });
}
```

在中大型的项目中，会有几个问题：
1. 将牵涉到外部行为的逻辑写在 Controller，造成 Controller 代码臃肿难以维护
2. 违反 SOLID 的单一职责原则：外部行为不应该写在 Controller
3. Controller 直接相依于外部行为，使得我们无法对 Controller 做单元测试

比较好的方式是使用 Service，使用的步骤如下：
1. 将外部行为注入到 Service
2. 在 Service 使用外部行为
3. 将 Service 注入到 Controlelr

`app\Services\EmailService.php`：
```php
<?php

namespace App\Services;
use Illuminate\Mail\Message;
use Mail;

/**
 * Class EmailService
 *
 * @package App\Services
 */
class EmailService
{
    /**
     * @var \Mail
     */
    protected $mailer;

    /**
     * 将相依的 Mailer 注入到 EmailService
     * EmailService constructor.
     *
     * @param $mailer
     */
    public function __construct(Mail $mailer)
    {
        $this->mailer = $mailer;
    }

    /**
     * 发送 Email的逻辑写在 send() 不是使用 Mail Facade，而是使用 $this->mailer
     * @param array $request
     */
    public function send(array $request)
    {
        $this->mailer->queue('email.index',$request,function(Message $message){
            $message->sender(env('MAIL_USERNAME'));
            $message->subject(env('MAIL_SUBJECT'));
            $message->to(env('MAIL_TO_ADDR'));
        });
    }
}
```

`app\Controllers\UserController.php`：
```php
<?php

namespace App\Http\Controllers;

use App\Services\EmailService;
use Illuminate\Http\Request;

/**
 * Class UserController
 *
 * @package App\Http\Controllers
 */
class UserController extends Controller
{
    /**
     * @var \App\Services\EmailService
     */
    protected $emailService;


    /**
     * @param \Illuminate\Http\Request $request
     */
    public function store(Request $request)
    {
        $this->emailService->send($request->all());
    }
}
```

从原本相依于 `Mail Facade` ，改成相依于注入的 `EmailService`。
改用这种写法有几个优点，如下：
1. 将外部行为写在 Service，解决了 Controller 代码臃肿的问题。
2. 符合 SOLID 的单一职责原则： 外部行为写在　Service ，没写在 Controller。
3. 符合 SOLID 的依赖反转原则：Controller 并非直接相依于 Service，而是将 Service 依赖注入进 Controller。

### 使用 PHP 写的逻辑
如根据用户购买数量，给予同步的折扣，可能我们会在 Controller 直接写 `if () { ... } else { ... }` 逻辑。

`app\Controllers\UserController.php`：
```php
public function index(Request $request)
{
    $number = $request->input('number');
    $price = 500;
    $discount = 1;
    if ($number == 1) {
        $discount = 1;
    } elseif ($number == 2) {
        $discount = 0.9;
    } elseif ($number == 3) {
        $discount = 0.8;
    } else {
        $discount = 0.7;
    }
    $total = $price * $number * $discount;

    return $total;
}
```

在中大型项目中，会有几个问题：
1. 将 PHP 写的业务逻辑直接写在 Controller ，造成 Controller 的代码臃肿难以维护
2. 违反了 SOLID 的单一职责原则：业务逻辑不应该写在 Controller
3. 违反了 SOLID 的单一职责原则：若未来想要改变折扣的写算法，都需要用到此 Method，也也就是说这个 Method 同时包含了计算折扣于计算加总的职责，因此违反了 SOLID 的单一职责原则
4. 直接写在 Controller 的逻辑无法被其他 Controller 使用

`app\Services\OrderService.php`：
```php
<?php

namespace App\Services;

/**
 * Class OrderService
 *
 * @package App\Services
 */
class OrderService
{
    /**
     * 计算折扣
     *
     * @param $number
     *
     * @return float
     */
    public function getDisCount($number)
    {
        switch ($number) {
            case 1:
                return 1.0;
                break;
            case 2:
                return 0.9;
                break;
            case 3:
                return 0.8;
                break;
            default:
                return 0.7;
        }
    }

    /**
     * 计算最后价格
     *
     * @param $number
     * @param $discount
     *
     * @return int
     */
    public function getTotal($number, $discount)
    {
        return 500 * $number * $discount;
    }
}
```

在 Controller 中调用代码，如下：
```php
<?php

namespace App\Http\Controllers;

use App\Services\OrderService;
use Illuminate\Http\Request;

/**
 * Class UserController
 *
 * @package App\Http\Controllers
 */
class UserController extends Controller
{
    /**
     * @var \App\Services\EmailService
     */
    protected $orderService;

    /**
     * UserController constructor.
     *
     * @param \App\Services\OrderService $orderService
     */
    public function __construct(OrderService $orderService)
    {
        $this->orderService = $orderService;
    }
    
    /**
     * @param \Illuminate\Http\Request $request
     *
     * @return int
     */
    public function index(Request $request)
    {
        $number = $request->input('number');
        $discount = $this->orderService->getDisCount($number);
        return $this->orderService->getTotal($number, $discount);
    }
}
```

将原本的 `if () { .. } else { .. }` 逻辑改写成使用 `OrderService`，`Controller` 变得非常干净，也达成原来 `Controller`
接受 `Http Request`，调用其他 `Class` 的责任。

改用这种写法的几个优点：
1. 将 PHP 写的业务逻辑写在　Service ，解决了 Controller 代码臃肿的问题
2. 符合 SOLID 的单一职责原则： 业务逻辑写在 Service，没写在 Controller
3. 符合 SOLID 的单一职责原则：计算折扣与计算加总分开在不同的 Method，且归属于 OrderService，而非 Controller
4. 符合 SOLID 的依赖反转原则： Controller 并非直接相依于 Service，而是将 Service 依赖注入进 Controller
5. 其他 Controller 也可以重复使用这段业务逻辑

## 总结
* 实际上会有很多 Service ，需要自行依照 **SOLID** 原则去判断是否该建立 Service
* Service 使得业务逻辑从 Controller 中解放，不仅更容易维护、更容易拓展、更容易重复使用且更容易测试

## 原文链接
* [如何使用Service？](https://oomusou.io/laravel/service/)
* [如何使用 Service 模式?](https://blog.csdn.net/qq_24935119/article/details/89658281)
* [项目地址](https://github.com/oomusou/Laravel51ServiceExternal_demo)
