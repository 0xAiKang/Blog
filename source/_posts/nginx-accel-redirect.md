---
title: Nginx Accel Redirect
date: 2023-11-15 16:29:24
tags: ["Nginx"]
categories: ["Nginx"]
---

当需要在网站上提供文件访问或下载时，如果直接提供文件所在位置的链接，不仅会暴露文件位置，还会存在安全风险。

这时，Nginx 的 X-Accel 就是一个非常有用的工具，它可以安全、高效地提供文件访问服务。

X-Accel 是 Nginx 提供的一种重定向机制，它可以在 Nginx 内部实现文件的访问，而不会直接暴露文件路径。这种机制可以提高安全性，避免了直接访问文件路径的风险，并且可以实现更多的功能，如权限控制和防盗链等。

<!-- more -->

## Nginx 配置
Nginx 的配置很简单，有两种方式：
* root
* alias

```nginx
# 当访问 /protected_files/myfile.tar.gz 这个地址时
# 最终会 /var/www/files/myfile.tar.gz 这个文件
location /protected_files {
  internal;
  alias /var/www/files;
}

# 当访问 /protected_files/myfile.tar.gz 这个地址时
# 最终会找到 /var/www/protected_files/myfile.tar.gz 这个文件 
location /protected_files {
  internal;
  root /var/www;
}
```

两种方式的结果是一样的，但需要注意区别。

还可以代理到另外一台服务器：
```nginx
location /protected_files {
  internal;
  proxy_pass http://127.0.0.2;
}
```

## PHP 代碼

配置好 Nginx 之后，并不是直接通过访问 `/producted_files` 来访问的，需要在响应头中增加一些“[特殊的头](https://www.nginx.com/resources/wiki/start/topics/examples/x-accel/#special-headers)”。

其中，最重要的是 `X-Accel-Redirect` 这个响应头，下面以 PHP 代码为例：
```php
<?php

public function download() {
    // ...
  
    $filename = "";
    header("X-Accel-Redirect:/downloadAlias/$filename");
    header("Content-Type: application/octet-stream");
    header("Content-Disposition: attachment; filename=$filename");
    exit();
};
```

在这段代码中，主要是设置了响应头信息，其中：
* `Content-Type` 为 `application/octet-stream`，表示这是一个二进制流文件，需要下载；
* `Content-Disposition` 表示文件的下载方式，attachment 表示文件需要下载，而不是在浏览器中打开；
* `X-Accel-Redirect` 表示内部重定向地址，即需要下载的文件的地址。

## 实际应用
X-Accel 的应用场景，除了文件下载。

还可以用于需要文件读取的场景，如果使用原生的 PHP 读取文件的方式 `readfile()`，会一次读取整个文件。

当需要读取一些大文件时，这是非常不友好的。

这是也可以使用到 `X-Accel`，它会返回状态码为 [206](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/206) 的响应，将一个大文件，可以分成 N 次获取。

使用 `X-Accel` 方式读取文件：
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231115094927.png)

使用 `readfile` 方式读取文件：
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231115102401.png)
通過 Nginx Access.log 可以清晰地看到，前者是分了多次读取文件的，而后者只有一次就读完了。

## 参考链接
* [Nginx X-Accel](https://www.nginx.com/resources/wiki/start/topics/examples/x-accel/#less-info)
* [Nginx X-Accel Explained](https://blog.horejsek.com/nginx-x-accel-explained/)