---
title: 谷歌 Google Play 内购支付
date: 2024-01-21 17:13:38
tags: ["PHP", "Tutorial"]
categories: ["PHP", "Tutorial"]
---

谷歌支付相较于苹果支付，会麻烦许多，需要配置和注意的东西比较多。

这篇笔记会把遇到的一些坑，一一列举出来，以下是正文。

<!-- more -->

## 支付流程

谷歌支付流程和苹果支付流程有很多相似的地方，下面是谷歌支付流程：
1. App 从 Google Play 获取产品信息
2. 用户选择需要购买/订阅的产品
3. App 发送支付请求到 Google Play
4. Google Play 处理支付请求，返回 Purchase Token 等信息
5. App 将 Purchase Token 发送到服务端
6. 服务端收到收据后，发送到 Google Play 验证收据的有效性
7. Google Play 返回收据的验证结果
8. 根据 Google Play 返回的结果，决定用户是否购买成功

## Google Play Developer Api 身份认证

在上面的过程中，需要请求 Google Play Developer Api，这里就需要解决一个问题：如何进行身份认证。

Google Play Developer Api 提供两种方式进行身份认证：
1. OAuth 2.0
2. 服务账号

### OAuth2.0
网上许多教程都是使用 OAuth2.0 方式授权 Api，虽然也是能实现，但是需要通过 Web 页面进行授权，因此并不适用于 App 的使用场景，后面会在另外一篇笔记中，详细记录如何通过该方式进行身份认证。

更多有关 OAuth2.0 的介绍，可以查看 Google 官方的说明——使用[OAuth 2.0 授权](https://developers.google.com/identity/protocols/oauth2?hl=zh-cn)。

### 服务账号
通俗点说，服务账号是一种特殊的账号，用于进行身份认证，使用它可以访问该服务账号有权访问的所有资源。

更多有关服务账号的介绍，可以查看 Google 官方的说明——[服务账号概览](https://cloud.google.com/iam/docs/service-account-overview?hl=zh-cn)。

下面将介绍如何创建服务账号以及授权 Google Play Developer Api。

正式开始之前，首先得有一个 Google 开发者账号，如果没有直接去[注册](https://play.google.com/console/signup)一个，不过需要注意的是，注册需要支付 25 美元的一次性注册费。

#### 在 Google Cloud Console 中启用 Google API
1. 转到[Google Cloud Console](https://console.cloud.google.com/welcome)
2. 从项目列表中选择项目或创建一个新项目
3. 转到[API & Services](https://console.cloud.google.com/apis/dashboard)，然后单击 Enable Apis and Services 按钮

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120100116.png)

4. 在[API Library](https://console.cloud.google.com/apis/library)中，搜索[Google Play Android Developer Api](https://console.cloud.google.com/apis/library/androidpublisher.googleapis.com)和[Google Play Developer Reporting](https://console.cloud.google.com/apis/library/playdeveloperreporting.googleapis.com) Api。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120100236.png)

5. 为项目启用这两个Api

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120100338.png)

#### 创建服务账号
1. 转到[Google Cloud Console](https://console.cloud.google.com/welcome) => [IAM 和管理](https://console.cloud.google.com/iam-admin/iam) => [服务帐户](https://console.cloud.google.com/iam-admin/serviceaccounts)
2. 单击**创建服务帐户**

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240119173030.png)

3. 输入服务帐户名称，复制其**ID**，然后单击**创建并继续**

4. 授予此服务账号访问权限
* **Pub/Sub 管理员**（启用[Google 开发者通知](https://documentation.qonversion.io/docs/google-developer-notifications)）

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240119180107.png)
* **监控查看器**（允许监控通知队列）

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240119180157.png)

5. 跳过最后一步并单击**完成**

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240119180250.png)

#### 创建并下载服务密钥
1. 转到Google Cloud Console 中的**服务帐户**部分，选择最近创建的密钥，选择**管理密钥**。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240119180719.png)

2. 单击**添加密钥**按钮，然后**创建新密钥**

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240119180833.png)

3. 在弹出窗口中，确保选择 JSON，然后单击**创建**，以生成 JSON 密钥并将其下载

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240119181021.png)

下载的密钥要保存好，后续会用到。

#### 授权对应用的访问

1. 导航至Google Play Console 中的**用户和权限部分**，然后单击**邀请新用户**。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120100617.png)

2. 输入之前创建的服务账号的邮箱地址，添加到应用程序，然后单击应用

3. 授予账户权限，然后单击页面底部的**邀请用户**，下一步会被引导至**用户和权限**部分，授予下图中勾选的权限即可。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120101000.png)

至此，服务账号的准备工作就全部做完了，但是这个服务账号还不能马上使用。

这是因为 Google 服务凭证最多需要 24 小时，才能与 Google Developer Api 正常工作。

在此期间，如果使用了该服务账号，Google Play Developer Api 可能会返回以下错误**当前用户没有足够的权限来执行请求的操作**。

此时需要做的事情就是，等待就好了。

刚开始接入 Google Play Developer Api 时，并不知道这个坑，毕竟过往几乎所有的配置，都是实时生效的。

所以就导致，总以为是自己的问题，哪里的什么权限没有给到。

查阅各种资料，联系Google 技术支持，在这个地方，耗费了大量时间。。。

#### 使用服务帐户进行身份验证
等待 24 小时之后，找到前面下载的 JSON 密钥文件。

可以使用 Google Api 官方提供的 [google-api-php-client](https://github.com/googleapis/google-api-php-client)，也可以使用 [Laravel In-App purchase](https://github.com/imdhemy/laravel-in-app-purchases)，进行身份验证。

这里以 Laravel In-App purchase 为例：

```php
use Imdhemy\Purchases\Facades\Subscription;

class IAP {
    public function iapSubscribe () {
      $subscriptionReceipt = Subscription::googlePlay()->id($params['product_id'])->token($params['purchase_token'])->get();
      
      // 谷歌订单号
      $orderId = $subscriptionReceipt->getOrderId();
      //购买状态，0. 付款待处理 1. 付款已收讫 2. 免费试用 3. 待推迟升级/降级
      $purchaseState = $subscriptionReceipt->getPaymentState();

      // 订阅的购买类型, 0. 测试（即通过许可测试帐号购买）1.促销（即使用促销代码购买）
      $purchaseType = $subscriptionReceipt->getPurchaseType();

      // acknowledgementState，是否已经确认，0. 尚未确认 1. 已确认
      $acknowledgementState = $subscriptionReceipt->getAcknowledgementState();

      // cancelReason，订阅被取消或未自动续订的原因，0. 用户取消订阅，1. 系统（例如由于结算问题）取消订阅 2. 订阅被替换为新订阅 3. 开发者已取消订阅

      if ($purchaseState == 1) { 
          // todo 购买成功业务逻辑
      }
  }
}
```

其中 `product_id` 是，用户选择的产品或者订阅项，`purchase_token` 则是App 发起支付之后，Google Play 返回给客户端的加密之后订单信息，服务端要做的就是，将完整的订单信息请求 Google Play Developer Api 解密出来。

更多字段含义解释，查看[官方文档](https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.products?hl=zh-cn#resource-representations)。

## Google Play RTDN
和苹果支付不一样的是，谷歌支付的通知，需要完成一些配置才能使用。

Google Play RTDN 也叫做Google 实时开发者通知，是利用[Google Cloud Pub/Sub](https://cloud.google.com/pubsub/) 发送实时通知，
无需通过轮询 Api，即可监听订阅状态变化。

要使用 Cloud Pub/Sub，必须拥有启用了 Cloud Pub/Sub 的 Google Cloud Platform 项目。

接收通知需要执行以下步骤：
1. 创建一个主题
2. 创建 Pub/Sub 订阅
3. 授予服务帐户向主题发布消息的权限
5. 为您的应用启用实时开发者通知

### 创建主题
要创建主题，可以使用Google 提供的[Guide me](https://console.cloud.google.com/cloudpubsub?walkthrough_id=pubsub_quickstart) 功能，该功能引导主题创建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120112906.png)

然后单击**创建主题**按钮。在**主题 ID**字段中，输入主题的唯一 ID，然后单击**保存**。

### 创建发布/订阅
要接收来自该主题的消息，需要创建该主题的订阅。要创建订阅，需要执行以下操作：
1. 显示刚刚创建的主题的菜单
2. 单击**创建订阅**按钮
3. 输入订阅的名称
4. 选择**Push to endpoint**作为传送类型
5. 在 **Endpoint URL** 中输入创建的[后端服务器的 URL](http://localhost:3000/laravel-iap-docs/docs/get-started/routing#generate-a-signed-url)（用来接收通知的地址）
6. 单击**创建**按钮

### 授予权限

Google Pub/Sub 要授予 Google Play 主题发布通知的权限，才能正常使用，需要执行以下操作：
1. 在 Google Cloud Console 中找到主题
2. 打开权限详细信息

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120114512.png)

3. 在页面右侧，单击**添加主体**按钮

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120114536.png)

4. 添加服务帐户 `google-play-developer-notifications@system.gserviceaccount.com`，并授予**Pub/Sub Publisher**角色。


![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120114644.png)

5. 单击保存，完成主题设置

### 启用订阅
做好以上配置之后，还需要启用订阅才能正常使用。

1. 转到[Google Play Console](https://play.google.com/console)
2. 点击左侧菜单，营业设定
3. 选择订阅设定为**已启用**
4. 在**主题名称**字段中，输入创建的主题的名称。格式应为`projects/{project_id}/topics/{topic_name}`
5. 点击保存更改

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240120145251.png)

可以点击传送测试通知按钮，测试通知是否正常发送到上面填写的通知地址。

---

至此，谷歌支付的流程就跑通了。

如果还遇到其他问题，去[Google 帮助中心](https://support.google.com/googleplay/android-developer/community)搜索，会比搜索引擎的结果更有效一些，说不定就能看到一些有用的回答。

## 参考链接
* [服务账号概览](https://cloud.google.com/iam/docs/service-account-overview?hl=zh-cn)
* [OAuth 2.0 授权](https://developers.google.com/identity/protocols/oauth2?hl=zh-cn)
* [JWT 错误代码解释](https://developers.google.com/identity/protocols/oauth2/service-account?hl=zh-cn#error-codes)
* [SubscriptionPurchase 字段描述](https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.subscriptions?hl=zh-cn)
* [Google Play 订阅生命周期](https://developer.android.com/google/play/billing/lifecycle/subscriptions?hl=zh-cn)
* [创建和配置产品](https://developer.android.com/google/play/billing/getting-ready#products)
* [Laravel In-App Purchases Docs](https://imdhemy.com/laravel-iap-docs/docs/intro)