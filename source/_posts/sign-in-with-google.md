---
title: 使用 PHP 接入 Google 登录
date: 2023-10-03 09:10:13
tags: ["PHP"]
categories: ["PHP"]
---

Sign In With Google 谷歌第三方登录，服务端 PHP 篇。

<!-- more -->

## 准备
首先需要准备一个 Google 账号。

然后进入到 [Google Cloud](https://console.cloud.google.com)，创建新的项目。

点击 New Project，创建一个新的项目：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231008160023.png)

创建完成之后，下一步完善应用信息：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231008160032.png)

如果需要用到一些服务，例如Google Map、Machine learning、Google Workspace 等等服务，可以从这里去启用：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231008160041.png)

下一步需要做的是，配置 Google OAuth consent screen：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231008160052.png)

选择用户类型，因为最终需要在生产环境中去使用，因此选择 External。

接下来就是按照提示，一步步，填写必要的信息。

创建好 OAuth consent screen 之后，下一步就可以创建 Credentials。

点击 Create Credentials，会出现一个下拉选项：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231008160103.png)
* API key：使用简单的 API 密钥来标识项目，以便检查配额和访问权限，
* OAuth Client ID：征得用户同意，让应用有权限访问用户数据
* Service account：利用机器人账号来实现服务器到服务器的应用级身份验证

因为接下来是要实现 Google 第三方登录，因此选择 OAuth Client ID。

依据客户端，选择对应的应用类型：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231008160143.png)

补充好相关信息，点击 Create 即可：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231008160121.png)

## 登录流程
核心逻辑如下：
1. 用户点击 Google 登录，客户端跳转 Google 登录授权
2. 授权成功之后，携带 code 以及 state，重定向回登录页
3. 客户端根据 code 以及 state，请求 Google  Api，换取 accessToken
4. 客户端将 accessToken 发送给服务端，服务端请求 Google Api 获取用户信息

## PHP 服务端接入
因为客户端可以是 Web、iOS、 Android，此处省略了，客户端如何获取 accessToken 的步骤。

拿到 accessToken 之后，就很简单了，直接请求 Google Api 即可拿到用户数据。

```php
try {
$userinfoEndpoint = 'https://openidconnect.googleapis.com/v1/userinfo';
$userinfoResponse = file_get_contents($userinfoEndpoint, false, stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => 'Authorization: Bearer ' . $accessToken
    ]
]));
// Google User Info
$googleUser = json_decode($userinfoResponse, true);

if (empty($googleUser)) {
    echo "Google authentication failed";
}
} catch (\Exception $e) {
    echo $e->getMessage();
}
```

## 参考链接
* [Google Authorization](https://developers.google.com/identity/authorization?hl=zh-cn)
* [Google Authentication](https://developers.google.com/identity/authentication?hl=zh-cn)