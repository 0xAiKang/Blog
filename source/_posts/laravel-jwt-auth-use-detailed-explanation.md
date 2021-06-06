---
title: Laravel jwt-auth 使用详解
date: 2021-06-06 17:11:15
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

通常后端在开发Api 应用时，会给客户端一个唯一Token 进行标示，获取Token 的方式有很多，这里以 JWT 为例，介绍其概念及使用方法。

<!-- more -->

## JWT 
[JWT](https://jwt.io/) 是 `JSON Web Token` 的缩写，是一个非常轻巧的规范，这个规范允许我们使用 JWT 在用户和服务器之间传递安全可靠的信息。

JWT 由头部（header）、载荷（payload）与签名（signature）组成，一个 JWT 类似下面这样：
```
{
    "typ":"JWT",
    "alg":"HS256"
}
{
    "iss":"http://larabbs.test",
    "iat":1515733500,
    "exp":1515737100,
    "nbf":1515733500,
    "jti":"c3U4VevxG2ZA1qhT",
    "sub":1,
    "prv":"23bd5c8949f600adb39e701c400872db7a5976f7"
}
signature
```

* 头部声明了加密算法；
* 载荷中有两个比较重要的数据，exp 是过期时间，sub 是 JWT 的主体，这里就是用户的 id；
* 最后的 signature 是由服务器生成的签名，保证了 token 不被篡改。

这三部分是分别用 `base64url` 进行编码，然后通过`.` 符号组合在一起，最后得到的token 大概是这样：
```
xxxxxx.yyyyyy.zzzzzz
```
x、y、z 部分分别代表了各自部位对应的信息。

> 注意⚠️：JWT 最后是通过 Base64 编码的，也就是说，它可以被翻译回原来的样子来的。所以不要在 JWT 中存放一些敏感信息。

### 思考题：
Token 既然会下发给客户端，那为什么不用保存一份在服务端？

这是因为，唯一的签名保存在服务端，所以无需担心Token 中的信息可能被篡改，清楚这一点之后，只需要验证Token 的合法性。

### Token 验证
有了 token 之后该如何验证 token 的有效性，并得到 token 对应的用户呢？

Laravel 为我们准备好了 `auth` 这个中间件：

* 获取客户端提交的 token
* 检测 token 中的签名 signature 是否正确
* 判断 payload 数据中的 exp，是否已经过期
* 根据 payload 数据中的 sub（用户 ID），取数据库中验证用户是否存在
* 上述检测不正确，则抛出相应异常

并且幸运的是，一些勤劳的人，已经帮我们完成了这部分工作。

## jwt-auth
[jwt-auth](https://github.com/tymondesigns/jwt-auth) 是 Laravel 和 lumen 下一个优秀 JWT 组件。

```shell
composer require tymon/jwt-auth
```

安装完成后，需要生成一个 JWT 的 secret，这个 secret 很重要，用于最后的签名，更换这个 secret 会导致之前生成的所有 token 无效。
```shell
php artisan jwt:secret
```
可以看到在 `.env` 文件中，增加了一行 `JWT_SECRET`。

发布配置文件：
```shell
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```
会在`config` 目录下生成一个`jwt.php` 的配置文件。

修改 `config/auth.php`，将 `api guard` 的 `driver` 改为 `jwt`。
```php
// 默认的 guard
'defaults' => [
    'guard' => 'web',
    'passwords' => 'users',
],

// 自定义 guard
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],

// provider 的作用是指定认证所需的数据表或者模型，推荐使用 eloquent
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\User::class,
    ],
    
    // 'users' => [
    //     'driver' => 'database',
    //     'table' => 'users',
    // ],
],
```

如果你使用默认的 User 模型来生成 token，那么该模型需要继承 `Tymon\JWTAuth\Contracts\JWTSubject` 接口，并实现接口的两个方法 `getJWTIdentifier()` 和 `getJWTCustomClaims()`。
```php
<?php

namespace App;

use Tymon\JWTAuth\Contracts\JWTSubject;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements JWTSubject
{
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }
    
    public function getJWTCustomClaims()
    {
        return [];
    }
}
```

`getJWTIdentifier` 返回了 User 的 id（用于生成 Token），`getJWTCustomClaims` 是我们需要额外在 JWT 载荷中增加的自定义内容，这里返回空数组。

打开Tinker，尝试生成一个 token：
```php
>>> $user = User::first();
>>> Auth::guard('api')->login($user);

=> "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9hdWN0aW9uLnNtaHN3LmxvY2FsIiwiaWF0IjoxNjIyOTY2OTM3LCJleHAiOjE2MjMwNTMzMzcsIm5iZiI6MTYyMjk2NjkzNywianRpIjoibVJKY2wzVWlOMURTQWg2WSIsInN1YiI6MSwicHJ2IjoiMThiMDU4NmY1NWY5YjVhYzc3NmY3MjU3ZTNiODdkMzY2ZjZjNWM3MSJ9.bKFU2T2b-L_nF6uiwb6gZm76aGcWraZ0Bo9O6Xz5Tqw"
```
除了上面介绍的这种基于用户实例，返回Token的方式，还有另外两种方式可以创建Token：

1. 基于账密参数
```php
$credentials = request(['email', 'password']); 
$token = auth()->attempt($credentials)
```

2. 基于模型中的用户主键 id
```php
$token = auth()->tokenById(1);
```

拿到Token 之后，有两种使用方法：

1. 加到 url 中：`?token=你的token`
2. 加到 authorization 或者 header 中，建议用后者，因为在 https 情况下更安全：`Authorization:Bearer 你的token`

`jwt-auth` 有两个重要的参数，可以在 `.env` 中进行设置：
* `JWT_TTL`：生成的 token 在多少分钟后过期，默认 60 分钟
* `JWT_REFRESH_TTL`：生成的 token，在多少分钟内，可以刷新获取一个新 token，默认 20160 分钟，即 14 天。

这里解释一下这两个参数是怎么回事：
* `token` 的过期时间是出于安全性考虑
* `token_refresh` 的过期时间是出于用户体验考虑

出于安全性考虑，不会给用户下发永久有效的token，用户需要每隔一段时间来用过期的token 来跟服务器换取一个新的 token。

打个比方：

> 你在食堂办理了一张饭卡，有效期是1个月，每个月初都要去食堂激活一次，以整明你还在学校念书。
如果超过3个月内都没有激活这张饭卡，则视为该名学生已经不在学校，如果3个月后这名学生回来食堂吃饭，需要重新办理饭卡

同样的道理转换到token，只是这个激活步骤不需要用户真的去操作，这个是我们来做的，全程用户都是无感的（这个是后面的无痛刷新 token 的内容）。

### jwt-auth 使用详解
Token 拿到之后，如何应用到项目中呢？

需要配合 `auth:api` 中间件使用，你肯定会觉得奇怪，这个中间件好像没有在任何地方定义，怎么就能使用？

打开`app\Http\Kernel.php`，可以看到默认的路由中间件列表：
```php
protected $routeMiddleware = [
    'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

可以发现 `auth` 就是第一个中间件的别名，但是 `auth:api` 又是哪里来的呢？

`api` 是 `auth` 的路由参数，指定了要使用哪个看守器，这里指定使用 `api` 看守器，也就是 `auth.php` 中配置的 `api` 守卫：

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'user',
    ],
],
```
所以`auth:api` 并不是哪里自定义的别名中间件。

如果直接使用`auth` 中间件，相当于使用 `auth.php` 中指定的 `defaults` 看守器。

```php
// 路由中使用
Route::middleware("auth.admin")->group(function () {
    // ...
});

// 控制器中使用
public function __construct()
{
    $this->middleware('auth:api', ['except' => ['login']]);
}
```

#### 常用方法

```php
// 尝试根据提供的凭证验证用户是否合法
public function attempt(array $credentials = [], $remember = false);
// 一次性登录，不记录session or cookie
public function once(array $credentials = []);
// 登录用户，通常在验证成功后记录 session 和 cookie 
public function login(Authenticatable $user, $remember = false);
// 使用用户 id 登录
public function loginUsingId($id, $remember = false);
// 使用用户 ID 登录，但是不记录 session 和 cookie
public function onceUsingId($id);
// 通过 cookie 中的 remember token 自动登录
public function viaRemember();
// 登出
public function logout();

// 判断当前用户是否登录
public function check();
// 判断当前用户是否是游客（未登录）
public function guest();
// 获取当前认证的用户
public function user();
// 获取当前认证用户的 id，严格来说不一定是 id，应该是上个模型中定义的唯一的字段名
public function id();
// 根据提供的消息认证用户
public function validate(array $credentials = []);
// 设置当前用户
public function setUser(Authenticatable $user);
```

## 参考链接
* [JWT 完整使用详解](https://learnku.com/articles/10885/full-use-of-jwt)
* [JWT 扩展具体实现详解](https://learnku.com/articles/10889/detailed-implementation-of-jwt-extensions#35808e)
* [jwt-auth Quick start](https://jwt-auth.readthedocs.io/en/develop/quick-start/)