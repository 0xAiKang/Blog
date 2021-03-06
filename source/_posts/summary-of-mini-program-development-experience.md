---
title: 小程序开发经验总结
date: 2021-05-12 21:09:23
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

通常对接小程序，为了加快开发速度，会直接使用 [EasyWeChat](https://github.com/overtrue/wechat) 这个扩展包进行开发，EasyWeChat 已经封装好了微信相关的接口，使用起来非常方便。

<!-- more -->

## 小程序登录流程
下图是微信官方提供的时序图：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210511161052.png)

整理成流程，大概就是：
1. 小程序调用 `wx.login()` 接口获取**临时登录凭证（code）**，这一步用户是无感知的，无需用户授权；
2. 小程序提交 `code` 到 开发者服务器；
3. 开发者服务器通过 `appid`、`appsecret` 和 `code` 请求微信接口，换取用户的 `session_key` 和 `openid`；
4. 开发者服务器根据 `openid` 查找到对应的用户，存入 `session_key`，然后为该用户生成 access_token （JWT）返回给小程序。
5. 有了 `access_token` 小程序就可以调用任意接口了。

注意这里的 `session_key` 是一个比较特殊的设计，是用户的 会话密钥，需要存储在服务器中，调用获取用户信息、获取微信用户绑定的手机号等微信接口时，需要用这个 会话密钥 才能解密获取相关数据。每次调用 `wx.login()` 之后，微信都会自动生成新的 `session_key` ，导致之前的 `session_key` 失效，所以在必要的时候再去调用 `wx.login()`，而且还要及时保存 `session_key` 到服务器，以备后续使用。

此段流程整理来自Laravel 社区的[《L04 Laravel教程-微信小程序从零到发布》](https://learnku.com/courses/laravel-weapp/2.0/small-program-login-detailed-solution/4933#0dfb2b)。

## 获取OpenID和SessionKey
清楚了小程序的登录流程之后，可以动手来获取`code`了。

### 创建小程序
这里建议使用最新版本的微信开发者工具，以免出现一些不必要的问题。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210511163706.png)

填入AppID，点击新建。

初始化的小程序无需做任何更改，只需要在`wx.login()` 下面增加一行`console.log(res)` 将结果打印在控制台中，然后重新编译，即可看到控制台中输出了 `code`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210512211535.png)

拿到`code` 之后，就可以获取`OpenID`、`SessionKey`了。

### 代码调试

为了方便调试，这里直接在 Laravel 的Tinker 中进行测试，以下代码逐行粘贴在 tinker 中：
```php
use EasyWeChat\Factory;
$config = [
    'app_id' => 'wx2b41f13e5e*****',
    'secret' => '92474ce5be69c4fb25392d6cfb******',
    'response_type' => 'array',
    'log' => [
        'level' => 'debug',
    ],
];
$app = Factory::miniProgram($config);
$app->auth->session('CODE');

// 正常输出如下：
[
     "session_key" => "nFpZ0gfHKOtYQ878enM*****",
     "openid" => "oN7jq1ejz5KQX5JtEiBsL*****",
]
```

其中`app_id` 和`secret` 需要开发者通过微信开发平台自行获取：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210510234405.png)

不填或者填入错误的`app_id` 和 `secret` 都会导致获取OpenID异常。

如果遇到异常，可以对照微信官方文档——[全局返回码](https://developers.weixin.qq.com/doc/offiaccount/Getting_Started/Global_Return_Code.html)进行排查分析。

### 登录凭证校验
拿到OpenID及 SessionKey 之后，下一步就可以进行解密了，这一步也是通过`EasyWeCaht` 来完成。

根据前面的时序图，可以得知，登录凭证校验需要用到以下参数：
* appid：iv
* appsecret：encryptedData
* code：上面拿到的SessionKey

最后一个都好理解，可是前面两个分别是什么鬼？

不着急，先打开微信开发者工具，在`app.js` 中，加入以下代码：
```javascript
// app.js
App({
  onLaunch() {
    // ...

    // 获取用户信息
    wx.getSetting({
      success: res => {
        if (res.authSetting['scope.userInfo']) {
          // 已经授权，可以直接调用 getUserInfo 获取头像昵称，不会弹框
          wx.getUserInfo({
            success: res => {
              console.log(res)  
            }
          })
        }
      }
    })
  },
  globalData: {
    userInfo: null
  }
})
```

保存之后，再次编译，查看控制台输出：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210513151643.png)

这两个就是我们需要的数据了，拿到之后，再次打开`Tinker`：

```
$app->encryptor->decryptData("SessionKey", "iv", "encryptedData")
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210513151926.png)

正常情况下，返回结果会包含当前登录用户的个人信息，我这里用的是测试号，因此并没有譬如手机号这类字段。

至此，小程序登录与微信接口服务的交互就告一段落了，获取到用户身份之后的逻辑就不用多说了。

上面小程序端的代码只是演示如何拿到需要的参数，实际开发并不建议用此方式直接写在`app.js` 中。