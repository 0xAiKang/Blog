---
title: 使用 PHP 接入 Stripe 支付
date: 2023-04-01 15:13:56
tags: ["PHP"]
categories: ["PHP"]
---

[Stripe](https://stripe.com/) 是一个聚合支付平台，可以安全、快速地处理包括信用卡、Apple Pay、Google Pay、支付宝等市面上各种支付方式。

<!-- more -->

Stripe 的文档很齐全，如果不知道选择何种支付方式，或者找不到对应支付方式文档在哪，可以看看 Stripe 的[建议](https://dashboard.stripe.com/setup/recommendations)。

## 支付方式

* [Stripe Checkout](https://stripe.com/docs/payments/checkout)：使用低代码的方式快速构建一个定制的支付页面。
* [Stripe Elements](https://stripe.com/docs/payments/elements)：使用些许代码添加预构建的自定义 UI 组件来接受多种支付方式。

## 收款
首先安装依赖包：
```bash
$ composer require stripe/stripe-php
```

### Web
这种方式是通过在网站上添加一个结账按钮，调用 Stripe Api 创建 [Checkout Session](https://stripe.com/docs/api/checkout/sessions/create)，将客户重定向到 Stripe Checkout。

![Web](https://b.stripecdn.com/docs-statics-srv/assets/overview.6a4ea4b380bea93a5be8a820f3eb7c35.gif)

这种方式适用于，线上（Web）收款，如果需要通过 App 进行收款，可以继续往下看 App 的收款方式。

服务端代码：
```php
public function createPayOrderByCheckout()
{
    $host = $this->request->host();

    $priceId = $this->request->post("price_id");
    if (!$priceId) {
        $this->error(__('请填写 Price Id'));
    }

    try {
        Stripe::setApiKey($this->stripeSecretKey);
        $checkoutSession = Session::create([
            'line_items' => [[
                'price' => $priceId,
                'quantity' => 1,
             ]],
            'mode' => 'payment',
            'success_url' => $host . '/success.html',
            'cancel_url' => $host . '/cancel.html',
        ]);

        $this->success("订单创建成功", [
            "url"   => $checkoutSession->url,
            "id"    => $checkoutSession->id
        ]);
    } catch (\Stripe\Exception\ApiErrorException $exception) {
        $this->error("订单创建失败：". $exception->getMessage());
    }
}
```

### App
这种方式是 App 端确认好商品信息，点击下单按钮，服务端调用 Stripe Api 创建支付订单，并返回支付信息给客户端，拉起支付，用户完成支付，通过设置的 Webhook 进行回调通知。

![App](https://b.stripecdn.com/docs-statics-srv/assets/ios-overview.9a8b762e060eb4be79a5abb237378498.png)

服务端创建支付订单 Api：
```php
public function createPayOrderByApp(array $orderInfo)
{
    try {
        $stripe = new StripeClient($this->stripeSecretKey);
      
        // 创建 PaymentIntent
        $paymentIntent = $stripe->paymentIntents->create([
             // 支付金额（单位为分）
            'amount'   => $orderInfo['amount'] * 100,
            'currency' => $currency,
            // 将 order_no 作为与Stripe 关联依据
            'metadata' => [
                'order_no' => $orderInfo['order_no']
            ],
        ]);

        // 返回支付信息给客户端，拉起支付
        $this->success("订单创建成功", [
            'client_secret'  => $paymentIntent->client_secret,  // PaymentIntent 客户端密钥
            'publishable_key' => $this->stripePublishableKey     // Stripe 公钥
        ]);
    } catch (ApiErrorException $exception) {
        $this->error("订单创建失败：". $exception->getMessage());
    }
}
```

服务端回调 Api：
```php
public function webhook()
{
    Stripe::setApiKey($this->stripePublishableKey);

    $payload = @file_get_contents('php://input');
    $event = null;

    try {
        // 回调参数验证
        $event = Event::constructFrom(
            json_decode($payload, true)
        );
    } catch(\UnexpectedValueException $e) {
        // webhook 解析请求失败
        http_response_code(400);
    } catch(\Stripe\Exception\SignatureVerificationException $e) {
       // webhook 签名验证失败
        http_response_code(400);
    }

    // 处理回调事件
    switch ($event->type) {
        case 'payment_intent.succeeded':
            // 收款成功
            $paymentIntent = $event->data->object;
        
            // 付款人ID：$paymentIntent->latest_charge;
            // 退款时，会用到

            // todo 变更订单状态
            break;
        
        case 'payment_intent.canceled':
            // 支付取消
        
            // todo 取消支付
            break;

        default:
            // 其他事件 https://stripe.com/docs/api/events/types
            break;
    }

    http_response_code(200);
}
```

退款 Api：
```php
public function refund($orderInfo)
{
    Stripe::setApiKey($this->stripeSecretKey);
    $refund = \Stripe\Refund::create([
        // 原始付款对象ID
        'charge' => $orderInfo['payer_id'],
        // 退款金额，单位为分
        "amount" => $orderInfo['payment_amount'] * 100
    ]);

    if ($refund->status === "succeeded") {
        return true;
    }

    return $refund->failure_reason;
}
```

## 参考链接
* [线上付款](https://stripe.com/docs/payments/accept-a-payment)
* [模拟付款，测试集成](https://stripe.com/docs/testing)