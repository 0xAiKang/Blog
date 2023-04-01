---
title: Facebook 第三方登录
date: 2023-03-06 17:32:42
tags: ["PHP"]
categories: ["PHP"]
---

Facebook 第三方登录记录。

<!-- more -->

## 准备

首先[注册](https://developers.facebook.com/)一个 Facebook 开发者账号。

成功登录之后，需要先[创建](https://developers.facebook.com/apps/create/)一个应用。

此时需要选择一个类型，因为是对接 Facebook 登录，因此选择『消费者』类型就好了。

点击下一步继续，填写完基本信息，点击创建应用即可。

为应用添加产品，选择 Facebook 登录：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230228153639.png)
添加完产品之后，还需要进行 OAuth 客户端授权设置，主要是设置有效 OAuth 跳转 URI。

控制面板下，可以看到应用编号和应用密钥，后面会用到。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230228155231.png)
## 登录流程

![Facebook 登录流程](https://img-blog.csdnimg.cn/20200423142735203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1eXV5YW5nNjY4OA==,size_10,color_FFFFFF,t_70)
核心逻辑如下：
1. 用户点击 Facebook 登录，客户端跳转 Facebook 登录授权
2. 授权成功之后，携带 code 以及 state，重定向回登录页
3. 客户端根据 code 以及 state，请求 Facebook  Api，换取临时 accessToken
4. 客户端将 accessToken 发送给服务端，服务端请求 Facebook Api 获取用户信息

## PHP 服务端接入
服务端接入，安装 facebook sdk：
```bash
composer require facebook/graph-sdk
```

登录 API：
```php
public function facebookLogin() {
    // 获取客户端的AccessToken
    $accessToken = "";
  
    $fb = new Facebook([
        'app_id' => Env::get('facebook.APP_ID'),
        'app_secret' => Env::get('facebook.APP_SECRET'),
        'default_graph_version' => 'v16.0',
    ]);

    // 设置Access Token
    $fb->setDefaultAccessToken($accessToken);

    // 从Facebook 获取用户对象，字段按需获取
    $response = $fb->get('/me?fields=name,email');
    $fbUser = $response->getDecodedBody();
    /**
    * fbUser
    * id：Your Facebook UserID
    * name：Your Facebook Username
    * email：Your Facebook Email
    **/
}
```

需要注意的是，因为 Facebook 需要科学上网才能访问，如果本身在墙内，通过 SDK 请求是无法请求成功的，会得到以下错误信息：

> Connection timeout after 10000 ms

此时有两种解决方案：
1. 通过境外的服务器进行调试
2. 本地科学上网的情况下，更改 Facebook SDK 源码，`vendor/facebook/graph-sdk/src/Facebook/HttpClients/FacebookCurlHttpClient.php` 97 line，增加：`CURLOPT_PROXY => "Your local agent"`

## 参考链接
* [Facebook 登录概览](https://developers.facebook.com/docs/facebook-login/overview)
* [Graph Api 入门指南](https://developers.facebook.com/docs/graph-api/get-started)
* [利用 JavaScript SDK 部署网页版 Facebook 登录](https://developers.facebook.com/docs/facebook-login/web)