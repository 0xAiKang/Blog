---
title: 使用组合重构支付模块
date: 2021-10-30 09:34:00
tags: ["PHP", "设计模式"]
categories: ["PHP", "设计模式"]
---

最近技术老大给我留了一个课后作业，扔给我一段代码后，让我写一下这段代码有什么问题，以及该如何解决这些问题。

<!-- more -->

源码如下：
```PHP
     /**
     * 余额充值
     */
    public function pay()
    {
        // 省略验参部分
        Db::startTrans();
        try {
            $data['order_no'] = pay_order_no($this->uId);
            $data['title'] = $recharge_package->title;
            $data['uid'] = $this->uId;
            $data['package_id'] = $package_id;
            $data['money'] = $recharge_package->money;
            $data['give_money'] = $recharge_package->give_money;
            $data['sum_money'] = $recharge_package->money+$recharge_package->give_money;
            $data['created_at'] = date('Y-m-d H:i:s');
            $data['updated_at'] = date('Y-m-d H:i:s');
            RechargeOrderModel::create($data);
            $result = $this->wechatPay($data);
            Db::commit();
            return $this->success($result);
        } catch (\Exception $exception) {
            Db::rollback();
            return $this->error($exception->getMessage());
            return $this->error('支付失败');
        }
    }

    /**
     * 小程序微信支付
     * @param $data
     */
    private function wechatPay($data)
    {
        $config = config('pay.wechat');
        $config['notify_url'] = request()->Domain(). '/api/v1.user/wechatNotify';
        $open_id = $this->request->param("open_id");
        $openId = isset($open_id) ? $open_id : UsersModel::where('uid',
            $data['uid'])->value('open_id');
        $order = [
            'out_trade_no' => $data['order_no'],
            'total_fee'    => $data['money']
            'total_fee'    => 1,
            'body'         => '余额充值',
            'openid'       => $openId,
        ];
        return Pay::wechat($config)->mini($order);
    }
```



## 我看到的问题

1. 支付部分不可复用。回调地址是硬编码，加上方法是私有的，当下一次其他业务需要微信支付时，只能将`wechatPay`方法手动再复制一份。
2. 耦合较高。`RechargeOrderMode::create`  与 `wechatPay` 接收的是同一个参数。前者属于系统业务，后者属于第三方服务。如果其他业务需要调用微信支付，那调用者就得提前知道`wechatPay` 这个方法所接收的`data` 参数具体是什么。
3. `wechatPay` 方法目前只支持小程序支付，如果需要对接扫码或者 App 支付，在此基础上扩展起来不方便。
4. 异常捕获到之后没有记录，而是直接抛出了，不利于问题排查，另外就是服务端具体的异常信息无需让用户知道，应该抛出之前处理一下。



## 解决方案

涉及支付的地方，通常都会有订单。

创建订单和支付很像，创建订单可以有购买套餐、余额充值、开通会员等，而支付则可以有小程序支付、扫码支付、App 支付等。

可以使用策略模式来解决这个问题。策略模式用于将一组算法移到一个独立的类型中，然后通过简单工厂模式获取对应的策略对象。

下面试着来重构一下。



开发流程可以大致分为以下三块：

1. 创建支付订单
2. 拉起支付
3. 回调

------------

### 创建支付订单

1. 创建一个抽象类 `PayOrderStrategy`，其中定义了抽象方法 `createOrder()`，其目的在于约束子类实现创建订单策略。
2. 创建一个`Context`类，显式调用另一个对象的方法来执行请求。`Context` 类并不负责创建订单，它将这个任务交给了 `PayOrderStrategy` 的实现类。
3. 创建一个获取策略的简单工厂，其作用在于根据不同的类型可以返回对应的策略对象。
4. 定义策略。根据实际业务场景将对应的算法（业务逻辑）写在具体的策略类中。



文件树状图如下：

```
 ├── PayOrder 									// 支付订单 Service
 │   ├── PayOrderContext.php    // 联系上下文（调用者通过 Context 使用策略）
 │   ├── PayOrderStrategy.php   // 定义抽象方法
 │   ├── PayOrderFactory.php    // 获取策略
 │   └── Strategy 							// 具体策略实现
 │       ├── VipStrategy.php    // 开通vip 策略
 │       ├── ...							  // 其他策略
 
```



可以看到，这种结构的一个优点是各个类的职责更加集中，`VipStrategy` 对象只负责创建 VIP 订单的策略，`PayOrderFactory`  只负责实例化对象，`PayOrderContext` 则只负责管理支付订单。

相比只使用继承，组合对象能够使代码更加灵活，因为对象能够以多种方式动态地组合来处理任务。

###  拉起支付

支付的核心逻辑和上面差不多。

文件树状图如下：

```
├──Payment												 
│   ├── PaymentContext.php 				 
│   ├── PaymentFactory.php 				 
│   ├── PaymentStrategy.php				 
│   └── Strategy				           // 支付策略
│       ├── Wechat			 					 // 微信支付策略
│           ├── AppStrategy.php    // App 支付
│           ├── MiniStrategy.php   // 小程序支付
│           ├── ScanStrategy.php   // 扫码支付
│           ├── ...	 						   // 其他策略
```



### 回调

回调就不用多说了。



## 具体实现

### 订单模块

`PayOrder/PayOrderStrategy.php`：

```php
<?php

namespace App\Services\PayOrder;

abstract class PayOrderStrategy
{
    abstract public function createOrder(array $data);
}
```



`PayOrder/PayOrderContext.php`：

```php
<?php

namespace App\Services\PayOrder;

class PayOrderContext
{
    private PayOrderStrategy $strategy;

    public function __construct(PayOrderStrategy $strategy)
    {
        $this->strategy = $strategy;
    }

    public function createOrder(array $data)
    {
        return $this->strategy->createOrder($data);
    }
}
```



`PayOrder/PayOrderFactory.php`：

```php
<?php

namespace App\Services\PayOrder;

use App\Exceptions\InvalidRequestException;
use App\Services\PayOrder\Strategy\VipStrategy;

class PayOrderFactory
{
    public static function getInstance($orderType)
    {
        $map = [
            "vip" => new VipStrategy(),
            // ...
        ];

        if (isset($map[$orderType])) {
            return $map[$orderType];
        }

        throw new InvalidRequestException("订单类型不存在");
    }
}
```



`PayOrder/Strategy/VipStrategy.php`：

```php
<?php

namespace App\Services\PayOrder\Strategy;

use App\Services\PayOrder\PayOrderStrategy;

class VipStrategy extends PayOrderStrategy
{
    public function createOrder(array $data)
    {
        /**
         * 1. 创建订单
         * 2. 异常捕获及事务回滚，如果创建失败，直接抛出
         * 3. 组装并返回支付参数
         */

        // 支付所需参数
				return [
            'out_trade_no' => "",
            'total_fee'    => "",
            'body'         => "",
            'openid'       => "",
            // VIP 支付回调地址
            'notify_url'   => "",
        ];
    }
}
```



### 支付模块

`Payment/PaymentStrategy.php`：

```php
<?php

namespace App\Services\Payment;

interface PaymentStrategy
{
    public function pay(array $order);
}
```



`Payment/PaymentContext.php`：

```php
<?php

namespace App\Services\Payment;

class PaymentContext
{
    private PaymentStrategy $strategy;

    public function __construct(PaymentStrategy $strategy)
    {
        $this->strategy = $strategy;
    }

    public function pay(array $order)
    {
        return $this->strategy->pay($order);
    }
}
```



`Payment/PaymentFactory`：

```php
<?php

namespace App\Services\Payment;

use App\Services\Payment\Strategy\Wechat\AppStrategy;
use App\Services\Payment\Strategy\Wechat\MiniStrategy;
use App\Services\Payment\Strategy\Wechat\ScanStrategy;
use Yansongda\Pay\Exceptions\BusinessException;

class PaymentFactory
{
    public static function getInstance($payType)
    {
        $map = [
            "mini" => new MiniStrategy(),
            "app"  => new AppStrategy(),
            "scan" => new ScanStrategy(),
        ];

        if (isset($map[$payType])) {
            return $map[$payType];
        }

        throw new BusinessException("微信支付类型错误");
    }
}
```



`Payment/Strategy/Wechat/MiniStrategy.php`：

```php
<?php

namespace App\Services\Payment\Strategy\Wechat;

use App\Services\Payment\PaymentStrategy;
use Yansongda\Pay\Pay;

class MiniStrategy implements PaymentStrategy
{
    public function pay(array $data)
    {
        $config = config('pay.wechat');
        $config['notify_url'] = env('HOST') . $data["notify_url"];
        $order = [
            'out_trade_no' => $data["order_no"],
            'total_fee'    => $data['price'],
            'body'         => $data["body"],
            'openid'       => $data["open_id"],
        ];
        return Pay::wechat($config)->miniapp($order);
    }
}
```



`Payment/Strategy/Wechat/AppStrategy`：

```php
<?php

namespace App\Services\Payment\Strategy\Wechat;

use App\Services\Payment\PaymentStrategy;

class AppStrategy implements PaymentStrategy
{
    public function pay(array $order)
    {
        // TODO: Implement pay() method.
    }
}
```

###  客户端调用

```php
public function pay(VipPackageRequest $request)
    {
        /**
         * 1. 验参
         * 2. 组装参数
         * 3. 创建VIP 套餐订单
         * 4. 支付
         */
        $params = $request->all();

        $data = [];
        try {
            $orderStrategy = PayOrderFactory::getInstance("vip");
            $order = (new PayOrderContext($orderStrategy))->createOrder($data);

            // $params["pay_type"] 可以是小程序支付、App 支付、扫码支付
            $paymentStrategy = PaymentFactory::getInstance($params["pay_type"]);
            $result = (new PaymentContext($paymentStrategy))->pay($order);

            return $this->success($result);
        } catch (\Exception $exception) {
					  throw new InternalException($exception->getMessage(), $exception);
        }
    }
```
这里仅仅只是就订单和支付模块，简单重构了下，仍有不足的地方，（比如现在需要加入支付宝或者银行卡支付）还需要不断完善。

实际项目往往比这个场景要复杂，需要考虑的东西更多。