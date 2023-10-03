---
title: sign in with apple
date: 2023-10-03 09:10:13
tags: ["PHP"]
categories: ["PHP"]
---

Sign In With Apple 苹果第三方登录，服务端 PHP 篇。

<!-- more -->

想要接入 Apple 第三方登录，有两种方式：
1. 验证 [identityToken](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api/authenticating_users_with_sign_in_with_apple)
2. 验证 [authorizationCode](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api/verifying_a_user)

两种方式都可以获取到用户授权的 Apple 信息，但是第一种方式相对简单一些。

## 验证 identityToken

登录流程图如下：

![](https://docs-assets.developer.apple.com/published/190373199a/sign-in-with-apple-2~dark@2x.png)

客户端（APP 端）登录成功之后，会拿到以下信息：

```json
Object {
  "authorizationCode": "c6a79a2b031f343459c9d8f54838e933e.0.rryyt.4EIeY2_SW6qw1fphfGIZ-A",
  "email": null,
  "fullName": Object {
    "familyName": null,
    "givenName": null,
    "middleName": null,
    "namePrefix": null,
    "nameSuffix": null,
    "nickname": null,
  },
  "identityToken": "xxxx.yyyy.zzzz",
  "realUserStatus": 1,
  "state": null,
  "user": "001883.d382005d8c7845a9a5402dd10c398265.0950",
}
```

其中有一个 `identityToken` 字段，这个字段的值其实就是一个 JWT，可以看到使用 `.` 进行分隔，分为`header`、`payload`、`signature` 三部分。

客户端需要把这个 `identityToken` 传给后端，后端进行验证。

JWT里的 `signature`部分是苹果使用私钥对其进行的签名，要验证这个签名，需要先获得苹果的公钥，而公钥可以通过[JWKSet.Keys](https://appleid.apple.com/auth/keys)（JSON Web Key Set）来转换获得。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231002165733.png)
如果请求成功，响应对象会包含三个 Apple 的公钥 [JWKSet.Keys](https://developer.apple.com/documentation/sign_in_with_apple/jwkset/keys)

该 API 会返回多个密钥，密钥的数量可能随时间而变化，从这组密钥中，选择具有匹配密钥标识符 ( kid) 的密钥来验证 Apple 颁发的任何 JSON Web 令牌 (JWT) 的签名。

因为需要对JWT的签名进行验证，因此需要使用到 [firebase/php-jwt](https://github.com/firebase/php-jwt) 第三方包。
```php

public function loginByApple()
{
    $identityToken = request()->post("identity_token");
    $params = request()->all();
    if (empty($identityToken)) {
        return $this->failed("Apple id token require");
    }

    try {
        // 调用苹果接口获取 JWKS
        $apiResponse = file_get_contents('https://appleid.apple.com/auth/keys');
        $jwkSet = json_decode($apiResponse, true);
        if (!is_array($jwkSet)) {
            throw new AppRuntimeException("Get JWK failed");
        }

        // 将 JWKS 转换为公钥
        try {
            $pubKeys = JWk::parseKeySet($jwkSet);
        } catch (\Exception $e) {
            throw new AppRuntimeException("Parse key set failed:" . $e->getMessage());
        }

        // 使用公钥对JWT进行检验
        try {
            $headers = new \stdClass(['E256']);
            $payload = JWT::decode($identityToken, $pubKeys, $headers);
        } catch (\Exception $e) {
            throw new AppRuntimeException("Decode JWT failed:" . $e->getMessage());
        }

        // todo 登录逻辑
    } catch(AppRuntimeException $e) {
        return $this->failed($e->getMessage());
    } catch (\Exception $e) {
        return $this->failed("Apple logged in failed");
    }
}
```
如果验证成功，`$payload` 的核心字段有以下内容：
* `iss`：发行者注册的声明标识了发行身份令牌的主体。由于 Apple 生成令牌，因此值为：`https://appleid.apple.com`
* `sub`：用户的唯一标识符
* `aud`：开发者帐户 Client.id
* `iat`：Apple 发布身份令牌的时间
* `exp`：标识身份令牌过期的时间
* `email`：用户的 Email，可能是用户的真实电子邮件地址或代理地址，取决于授权时，是否隐藏真实电子邮箱。
* `email_verified`：是否验证 Email
* `is_private_email`：是否是代理地址

拿到用户的 Apple ID 和Email 之后，就可以完成后续的登录逻辑了。

## 验证 authorizationCode
登录流程如下：

![](https://docs-assets.developer.apple.com/published/bdfb4885e6/sign-in-with-apple-3~dark@2x.png)

第二种方式验证时，所需要的东西多一些：
* client_id：Apple 开发者后台的App ID 或 Services ID
* client_secret：由开发者生成的 JWT 令牌，验证 Authorization code 和刷新令牌需要用到该参数。

> 关于如何创建 Client Secret，可以[点击查看](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens#3262048)。

更多其他请求参数，可以[查阅更多](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens#http-body)。


## 参考链接
* [Sign in with Apple REST API](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api)
* [Sign in with Apple JS](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_js)
* [Authenticating users with Sign in with Apple](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api/authenticating_users_with_sign_in_with_apple)
* [Verifying a user](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api/verifying_a_user)
* [How to Sign in with Apple](https://sarunw.com/tags/sign%20in%20with%20apple/)