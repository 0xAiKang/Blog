---
title: 使用 PayPal-PHP-SDK 接入 PayPal 支付
date: 2023-03-19 14:20:01
tags: ["PHP"]
categories: ["PHP"]
---

最近业务需要，需要接入国外的第三方支付——PayPal。

<!-- mroe -->

因为一些原因，没有使用 PayPal 官方推荐的 Braintree 聚合支付，而是使用原生的 Rest Api。

官方为 Rest Api 封装了一个 SDK，虽然已经很久没有更新了，不过好在做了向下兼容，只是版本低一些，好在还能用。

在正式介绍之前，先来了解一下 PayPal 支付的流程：
1. 在 [PayPal 管理中心](https://developer.paypal.com/home) 创建一个开发者账号，获取 API 初始凭证信息，包括 Client ID 和 Secret
2. 使用 Client ID 和 Secret 创建 PayPal 实例，并创建支付请求对象，设置支付金额、货币类型、付款描述、同步回调地址等信息
3. 调用 PayPal 客户端实例的 `execute()` 方法，发送支付请求
4. 根据支付结果，跳转到相应成功或者失败的页面
5. 设置 Webhooks，如果支付成功，等待 PayPal 异步回调通知支付结果

这个支付流程，和平常见到的支付宝支付流程，有所区别。

支付宝的支付流程是，只需要获取支付 url，然后去支付宝网站里面完成支付，最后异步通知。

PayPal 支付则有所区别，PayPal 返回的 approval_url 只是获取用户授权，最终还是需要回到自己的网站再一次请求 PayPal 进行支付，最后才是异步通知。

## 创建支付订单

通过 Composer 安装SDK：
```bash
$ composer require paypal/rest-api-sdk-php
```

创建支付订单：
```php
class PaypalStrategy implements PaymentStrategy
{、
    private $clientId;

    private $clientSecret;

    private $environment;

    private $currency;

    private $apiContext;

    private $redirectUrl;

    public function __construct()
    {
        $this->clientId = Env::get("paypal.CLIENT_ID");
        $this->clientSecret = Env::get("paypal.CLIENT_SECRET");
        $this->environment = Env::get("paypal.ENVIRONMENT");
        $this->currency = "USD";
        $this->redirectUrl = "http://".Env::get("app.app_url") . "/api/order/paypalExecute";

        $this->apiContext = new ApiContext(
            new OAuthTokenCredential(
                $this->clientId,
                $this->clientSecret
            )
        );
      
       if ($env === "production") {
            $this->apiContext->setConfig([
                'mode' => 'live',         // 设置为生产环境
            ]);
        }
    }
    
    // 支付
    public function pay(array $orderInfo, array $paymentParams)
    {
        $payer = new \PayPal\Api\Payer();
        $payer->setPaymentMethod('paypal');

        $item = new Item();
        $item->setName("商品名称")
            ->setCurrency($this->currency)
            // 设置数量
            ->setQuantity(1)
            ->setPrice($orderInfo['paymentamount']);

        $itemList = new ItemList();
        $itemList->setItems([$item]);

        $amount = new \PayPal\Api\Amount();
        // 设置总价
        $amount->setTotal($orderInfo['paymentamount'])
            ->setCurrency($this->currency);

        $transaction = new \PayPal\Api\Transaction();
        $transaction->setAmount($amount)
            ->setItemList($itemList)
            ->setDescription("商品描述")
            ->setInvoiceNumber($orderInfo['orderno']);

        // 设置授权成功之后，重定向地址
        // 这个地址很重要，是 PayPal 授权成功之后，同步回调获取付款人 ID
        $redirectUrls = new RedirectUrls();
        $redirectUrls->setReturnUrl($this->redirectUrl . "?success=ture")
            ->setCancelUrl($this->redirectUrl . "?success=false");

        $payment = new Payment();
        $payment->setIntent('sale')
            ->setPayer($payer)
            ->setTransactions(array($transaction))
            ->setRedirectUrls($redirectUrls);

        try {
            // 创建交易
            $payment->create($this->apiContext);
            return [
                // Paypal payment id
                "third_order_no" => $payment->id,
                // Paypal 授权地址
                "approval_url"   => $payment->getApprovalLink(),
            ];
        } catch (\PayPal\Exception\PayPalConnectionException $exception) {
            throw new \Exception($exception->getData());
        }
    } 
}
```

## 支付同步回调

```php
// 同步支付回调（用于获取付款人 ID）
public function execute($payload)
{
    // 是否取消支付标识
    $success = $payload['success'] ?? false;
    // 支付ID
    $paymentId = $payload['paymentId'] ?? "";
    // 支付人ID
    $payerId = $payload['PayerID'] ?? "";

    if (empty($paymentId) || empty($payerId)) {
        // 支付失败
        return "fail";
    }

    if (false === $success) {
        // 取消支付
        
        return "cancel";
    }

    $payment = Payment::get($paymentId, $this->apiContext);
    $execute = new PaymentExecution();
    // 设置付款人 ID
    $execute->setPayerId($payerId);
    try {
        // 确认支付（真正发起支付请求的地方）
        $payment->execute($execute, $this->apiContext);
    } catch (\Exception $e) {
        // 支付失败
        return "fail";
    }
    // 支付成功
    return "success";
}
```

授权完成之后，PayPal 会重定向至 redirectUrl，并携带 paymentId 和PayerID，拿到这两个参数之后，最后发起支付请求。

## 支付异步回调
PayPal 支付异步回调，主要是两个 Event Type：
* `PAYMENTS.PAYMENT.CREATED`：表示支付创建完成，即支付请求已经被创建并准备好向用户展示。这个 Event Type 触发的条件是：当 PayPal 收到一个支付请求并且准备好向用户展示时，会向 Webhooks 发送一个 `PAYMENTS.PAYMENT.CREATED` 的事件。
* `PAYMENT.SALE.COMPLETED`：表示销售完成，即支付已经完成并且相关的款项已经被转移。这个 Event Type 触发的条件是：当 PayPal 支付完成并且相关款项已经被转移时，会向 Webhooks 发送一个 `PAYMENT.SALE.COMPLETED` 的事件。

```php
// 异步支付回调
public function webhook($payload)
{
    $payload = json_decode($payload, true);
    if (empty($payload)) {
        // 回调参数解析失败
        return false;
    }

    // todo 验签
  
    try {
        // 详细回调参数，参见 https://developer.paypal.com/api/rest/webhooks/#link-samplemessagepayload
        
        switch ($payload['event_type']) {
            // 支付订单创建成功
            case "PAYMENTS.PAYMENT.CREATED":
            
                break;
            
            // 买家完成支付
            case "PAYMENT.SALE.COMPLETED":
                // 记录买家 ID，后面退款会用到
                break;
        }
        // todo 业务逻辑处理
        
        return true;
    } catch (\Exception $e) {
        // 订单回调处理异常

        return false;
    }
}
```

## 退款

```php
// Paypal 退款
public function refund($orderInfo)
{
    try {
            $amount = new Amount();
            $amount->setCurrency($this->currency)
                ->setTotal($orderInfo['payment_amount']);

            $refundRequest = new RefundRequest();
            $refundRequest->setAmount($amount);

            $sale = new Sale();
            // 付款人 ID
            $sale->setId($orderInfo['payer_id']);

            $result = $sale->refundSale($refundRequest, $this->apiContext);
        } catch (\Exception $e) {
            // 退款失败
            return $e->getMessage();
        }
        // 退款完成
        return true;
}
```
退款需要用到付款人 ID，可以在支付异步回调时，保存下来。

退款记录：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230318100509.png)
## 总结

上面的示例代码，并不能放到项目中使用，需要调整部分参数。

使用 PayPal 的沙盒账号，登录到[Sandbox](https://www.sandbox.paypal.com/myaccount/home)，可以看到交易记录及退款记录：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230316120925.png)

如果有其他技术问题，可以[提交工单](https://www.paypal-support.com/)。

## 参考链接
* [PayPal Payment Api V1](https://developer.paypal.com/docs/api/payments/v1/)
* [Payment->execute gives error payer_id Value too long (Max len 20)](https://github.com/paypal/PayPal-PHP-SDK/issues/120)
* [PayPal PHP SDK](https://github.com/paypal/PayPal-PHP-SDK)
* [Rest Api Developer Docs](https://developer.paypal.com/api/rest/)
* [PayPal PHP SDK Docs](http://paypal.github.io/PayPal-PHP-SDK/)
* [手动发起 Webhooks 通知](https://developer.paypal.com/dashboard/webhooksSimulator)