---
title: PHP 中使用 hash_hmac 加密
date: 2020-07-23 00:18:45
tags: ["PHP", "Hash", "Node"] 
categories: ["PHP"]
---

今天做项目时，遇到一个问题，需要将一段哈希值按照某种规则进行加密。源码是用`Node`写的，需要翻译成`PHP` 版本的。

<!-- more -->

### PHP中使用 Hmac 方法生成带有密钥的哈希值

在Node.js 中，这是一段用于生成“加盐”的哈希值。
```
var crypto = require('crypto');

var secret = "122410"
var key = "key"
var hash = crypto.createHmac('sha256', secret).update(key).digest('hex')

console.log(hash);
// dcc9ddf4836d4ecb6bd12fccc983207f39cfb84c43c01932eee22357cf0567b4
```

如果要翻译成PHP版本，其实非常简单，直接使用PHP 的 `hash_hmac`函数就可以了。

```
<?php
$secret = "122410"; 
$key = "key";
echo hash_hmac("sha256", $key, $secret);
// dcc9ddf4836d4ecb6bd12fccc983207f39cfb84c43c01932eee22357cf0567b4
```

#### 将密钥设置成二进制
如果需要加密的部分，并不是普通的字符串，而是二进制字符串，那么需要使用`pack`函数。

```
var_dump(hash_hmac("sha1", "office:fred", "AA381AC5E4298C23B3B3333333333333333333"));

// 5e50e6458b0cdc7ee534967d113a9deffe6740d0
// 预期结果：46abe81345b1da2f1a330bba3d6254e110cd9ad8
```

先将十六进制字符串转换为二进制数据，然后再将其传递给`hash_hmac`：
```
var_dump(hash_hmac("sha1", "office:fred", pack("H*", "AA381AC5E4298C23B3B3333333333333333333")));

// 46abe81345b1da2f1a330bba3d6254e110cd9ad8
```

### Node中使用crypto进行md5 加密
在PHP 中，如果需要获取某个字符串的md5 加密之后的哈希值，非常简单，直接使用`md5` 函数即可。

但是在`node.js` 中，并没有为我们直接提供这样的函数，所以需要手动调用`crypto` 模块去转换：
```
var pwd = "122410";
var hash = crypto.createHash('md5').update(pwd).digest('hex');

// 913975c2f972ba6bbf5ba593c68a5dc5
```

### 参考链接
* [如何在PHP中将hmac sha1密钥设置为十六进制？](https://stackoverflow.com/questions/13012239/how-to-set-the-hmacsha1-key-to-hex-in-php)
* [在线转换工具](https://caligatio.github.io/jsSHA/)
* [php hash_hmac 函数](https://www.php.net/manual/zh/function.hash-hmac.php)
* [node.js crypto 模块](http://nodejs.cn/api/crypto.html)