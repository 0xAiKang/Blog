---
title: Apple In-App Purchase 支付（2）
date: 2024-01-19 17:54:32
tags: ["PHP", "Tutorial"]
categories: ["PHP", "Tutorial"]
---

苹果的官方支付文档比较详细，但如果想要在短时间内接入好完整流程，也是有一定难度的。

好在有一些勤劳的人已经为我们完成了艰苦的工作——[Laravel In-App purchase](https://github.com/imdhemy/laravel-in-app-purchases)。

使用 Laravel In-App purchase 这个扩展包，可以很轻松接入苹果支付。

<!-- more -->

## 安装

通过 Composer 安装：
```bash
$ composer require imdhemy/laravel-purchases
```

发布配置文件：
```bash
$ php artisan liap:config:publish
```

`config/liap.php` 将创建一个包含以下配置键的文件，核心配置项如下：
* `routing`: 允许添加自定义路由配置
* `google_play_package_name`: Google Play 包名称
* `appstore_password`: App Store 共享密钥，下面会介绍如何获取
* `eventListeners`: 事件列表

**仅当**需要从App Store请求测试通知时，才需要以下键：
* `appstore_private_key_id`：来自 App Store 连接的私钥 ID（例如：2X9R4HXF34）
* `appstore_private_key`：私钥文件的路径（例如：/path/to/SuperSecretKey\_ABC123.p8）
* `appstore_issuer_id`：App Store Connect 中“密钥”页面中的颁发者 ID（例如：57246542-96fe-1a63-e053-0824d011072a）
* `appstore_bundle_id`：应用程序的捆绑 ID（例如：com.example.testbundleid2021）

## 创建销售产品
根据实际情况，选择创建对应类型的销售产品，订阅、消耗品还是非消耗品。

要在应用内提供应用内购买，需要先在 App Store Connect 中添加其信息：
1. 前往[App Store Connect](https://appstoreconnect.apple.com/)
2. 从**我的应用程序**中，选择应用程序
3. 在侧边栏中的**功能**下，选择**应用内购买**
4. 单击添加按钮 (+)
5. 选择**消耗品**、**非消耗品**、还是**订阅**
6. 通过添加参考名称和产品 ID 来填写表格
7. 单击**创建**按钮

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240105094803.png)

## 获取 App Store 凭证
请求 App Store Api 时，需要用到 **App-Specific Share Secret**，通过以下方式获取：

1. 前往[App Store Connect](https://appstoreconnect.apple.com/)
2. 找到**我的应用程序**并选择要配置的应用程序
3. 从**左侧菜单**“常规”部分下选择**应用程序信息**
4. 从**右侧的**“应用程序特定共享密钥”部分下选择**“管理”**

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240113155526.png)

6. 生成并复制共享秘密

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240113155657.png)


## 支付流程

支付流程：
1. App 从 App Store 获取产品信息
2. 用户选择需要购买/订阅的产品
3. App 发送支付请求到 App Store
4. App Store 处理支付请求，返回 transaction 信息
5. App 将transaction receipt 发送到服务器
6. 服务器收到收据后，发送到 App Stroe 验证收据的有效性
7. App store 返回收据的验证结果
8. 根据 App store 返回的结果，决定用户是否购买成功

流程图如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240102165641.png)
这个支付流程和一些主流的支付流程不一样，传统的支付流程是在服务端发起支付，然后通过回调确认是否支付成功。

而 In-App purchase 的核心支付流程是由，客户端发起支付，然后再由服务端去确认是否支付成功。

上面流程中，大部分都是 App 上的逻辑，就不在此过多介绍。

## 验证订单信息

从第六步开始，才与服务端有关，需要由服务端，向 App Store 发起验证。

```php
use Imdhemy\Purchases\Facades\Subscription;

class OrderController
{

    public function iapSubscribe()
    {
          $params = request()->all();
      
          $receiptResponse = Subscription::appStore()->receiptData($params['receiptData'])->verifyReceipt();
          $receiptStatus = $receiptResponse->getStatus();
      
          // 订阅成功
          if ($receiptStatus->isValid()) {
                $latestReceiptInfo = $receiptResponse->getLatestReceiptInfo();
                // 获取到最新一条订阅记录
                $receiptInfo = $latestReceiptInfo[0];

                // 产品 ID
                $productId = $receiptInfo->getProductId();
                // 交易 ID
                $transactionId = $receiptInfo->getTransactionId();
                // 原始交易 ID（一个Apple ID 的原始交易ID，始终是一样的）
                $originalTransactionId = $receiptInfo->getOriginalTransactionId();
                // 订阅到期时间
                $expiresDate = $receiptInfo->getExpiresDate()->toCarbon()->toDateTimeString();
                    
          }
          // 订阅失败
    }
}
```

App 那边会从 Apple 那边拿到一个 receiptData 收据，这个收据用于向App Store，验证订单信息，是否有效。

## 通知
除了处理收据问题，另外一个比较重要的就是通知了。

当订阅过期时，会发送过期对应的事件、当用户重新续订时，会发送续订事件、当订阅状态发生变化时，也会发送对应事件。

常用的订阅事件及子事件：
* SUBSCRIBED：订阅成功
  * INITIAL_BUY: 首次购买
  * RESUBSCRIBE: 重新订阅
* DID_RENEW: 订阅续订
   * BILLING_RECOVERY：之前未能续订的过期订阅已成功续订
   * : 活动订阅已成功自动续订新的交易期
* DID_CHANGE_RENEWAL_STATUS: 订阅状态变更
   * AUTO_RENEW_ENABLED: 重新启用订阅自动续订
   * AUTO_RENEW_DISABLED: 禁用了订阅自动续订
* EXPIRED: 订阅过期
   * VOLUNTARY: 订阅在用户禁用订阅续订后过期
   * BILLING_RETRY: 订阅过期，因为计费重试期结束，账单交易没有成功
   * PRICE_INCREASE: 订阅过期，因为用户不同意需要用户同意的价格上涨
   * PRODUCT_NOT_FOR_SALE: 订阅过期，因为在订阅尝试续订时，该产品无法购买

如果要查看更多的订阅事件，可以查看[App Store 订阅事件列表](https://developer.apple.com/documentation/appstoreservernotifications/notificationtype)。

```php

use Imdhemy\AppStore\Jws\Parser;
use Imdhemy\AppStore\ServerNotifications\V2DecodedPayload;

class Notification
{

  public function handle() {
    $params = $request->all();
    
    // App Store 通知参数
    $jws = Parser::toJws($params['signedPayload']);
    // 解析通知参数
    $payload = V2DecodedPayload::fromJws($jws);
    
    // 通知类型
    $notificationType = $payload->getType();
    // 原始订单 ID
    $originalTransactionId = $payload->getTransactionInfo()->getOriginalTransactionId();
    // App Store 产品ID
    $productId = $payload->getTransactionInfo()->getProductId();
    // 当前订单 ID
    $transactionId = $payload->getTransactionInfo()->getTransactionId();
    $type = $payload->getTransactionInfo()->getType();
  }
}
```

正常解析出来，可以得到以下信息：
```json
{
    "quantity":"1",
    "product_id":"moodmusics_8_1w",
    "transaction_id":"2000000480401937",
    "original_transaction_id":"2000000480401937",
    "purchase_date":"2023-12-15 07:27:31 Etc\/GMT",
    "purchase_date_ms":"1702625251000",
    "purchase_date_pst":"2023-12-14 23:27:31 America\/Los_Angeles",
    "original_purchase_date":"2023-12-15 07:27:38 Etc\/GMT",
    "original_purchase_date_ms":"1702625258000",
    "original_purchase_date_pst":"2023-12-14 23:27:38 America\/Los_Angeles",
    "expires_date":"2023-12-15 07:30:31 Etc\/GMT",
    "expires_date_ms":"1702625431000",
    "expires_date_pst":"2023-12-14 23:30:31 America\/Los_Angeles",
    "web_order_line_item_id":"2000000044675530",
    "is_trial_period":"true",
    "is_in_intro_offer_period":"false",
    "in_app_ownership_type":"PURCHASED",
    "subscription_group_identifier":"21418618"
}
```
关于字段完整解释，可以查看 [Transaction data types](https://developer.apple.com/documentation/appstoreservernotifications/responsebodyv2/responsebodyv2decodedpayload/transaction_data_types)。

## 配置通知地址

Laravel In-App purchase 提供 URL 在 App Store Connect / Google Play 中设置服务器通知地址。

1. 使用 `php artisan liap:url` 命令，可以通过 `routing.signed` 配置选择是否需要带有签名的 URL：
```bash
$ php artisan liap:url
Select provider:
  [0] All Providers
  [1] App Store
  [2] Google Play
 > 0

 Signed routes are disabled. Do you want to generate signed routes? (yes/no) [no]:
 > yes
 
+-------------+--------------------------------------------------------------------------------+
| Provider    | URL                                                                            |
+-------------+--------------------------------------------------------------------------------+
| App Store   | http://localhost/liap/notifications?signature=<signature>&provider=app-store   |
| Google Play | http://localhost/liap/notifications?signature=<signature>&provider=google-play |
+-------------+--------------------------------------------------------------------------------+
```

签名用于验证，请求是否合法。

2. 登录[App Store Connect](https://appstoreconnect.apple.com/)并选择应用程序
3. 在**应用程序信息** > **应用程序商店服务器通知**部分下，将通知 URL 粘贴到**生产服务器 URL**和 **沙盒服务器 URL**字段中
4. 在**版本**字段中选择版本`1`或`2`

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240113144708.png)

版本不同，会导致通知类型有所不一样，完整的 [notificationType](https://developer.apple.com/documentation/appstoreservernotifications/notificationtype) 可以查看苹果开发者文档。

关于苹果支付，因为跟传统的支付流程不太一样，内容还是挺多的，暂时只用到订阅部分，后面有其他的内容再补充。

## 参考链接
* [Enter server URLs for App Store Server Notifications](https://developer.apple.com/help/app-store-connect/configure-in-app-purchase-settings/enter-server-urls-for-app-store-server-notifications/)
* [App Store Server Notifications](https://developer.apple.com/documentation/appstoreservernotifications)
* [Validating receipts with the App Store](https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/validating_receipts_with_the_app_store)
* [Laravel In-App purchase Doc](https://imdhemy.com/laravel-iap-docs/docs/intro)