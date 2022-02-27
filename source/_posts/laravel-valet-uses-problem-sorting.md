---
title: Laravel Valet 使用问题整理
date: 2022-02-27 14:48:47
tags: ["PHP", "Laravel", "Mac"]
categories: ["PHP", "Laravel", "Mac"]
---

[Valet](https://laravel.com/docs/9.x/valet) 是 Mac 极简主义者的 Laravel 开发环境。

<!-- more -->

## 问题一

`brew services stop php`和 `valet stop` 全部停止之后：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220227111807.png)

访问站点：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220227111905.png)

`brew services stop php` 之后，启用 `valet start` ：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220227111929.png)

站点还是访问不了，一直显示 502，同时`~/.config/valet/Log/nginx-error.log` 会输出以下日志：

```
2022/02/22 20:32:00 [crit] 36873#0: *1 connect() to unix:/Users/boo/.config/valet/valet.sock failed (13: Permission denied) while connecting to upstream, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", upstream: "fastcgi://unix:/Users/boo/.config/valet/valet.sock:", host: "localhost"
```

查了一下，看到了[这个答案](https://github.com/laravel/valet/issues/269#issuecomment-446464150)，最后尝试以下命令，解决：
```shell
composer global update
valet install
valet restart
```

## 问题二

再次访问默认站点，即可看到正常输出页面：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220227111958.png)

但是当我访问同目录下的其他非默认站点时（valet 的顶级域名，比如 phpinfo.test），又会出现 502，这是为什么呢？\
而且这一次还没有看到任何`error.log` 输出。

那这个就和`Nginx` 或者 `PHP-FPM` 没有什么关系了。

这是因为本地的Clash 设置了系统代理，所有请求都是通过 `127.0.0.1:7890` 转发出去的。

具体可以看一下下面两个Issue：
1. [打开clashx之后，mac系统配置的hosts就会失效](https://github.com/Dreamacro/clash/issues/423)
2. [开启系统代理某些情况下会导致无法正常连接](https://github.com/Fndroid/clash_for_windows_pkg/issues/1175)

## 总结
1. 启用 Valet 的话，就不用再手动使用`brew` 启用PHP 或者 Nginx 了，否则端口会冲突
2. 本地启用系统代理的情况下，访问Valet 下的顶级域名站点，通常会失败

## 常用命令

Valet 常用命令

|命令	|描述 |
| ------- | ------- |
|valet forget |	从一个『驻留』目录运行此命令，从驻留目录列表将其它移除|
|valet log | 从 Valet 的服务中查看日志 |
|valet paths|	查看所有『驻留』路径|
|valet restart	|重启 Valet 守护进程|
|valet start	|开启 Valet 守护进程|
|valet stop|	停止 Valet 守护进程|
|valet trust|	为 Brew 和 Valet 添加文件修改权限使 Valet 输入命令的时候不需要输入密码|
|valet uninstall	|完成卸载 Valet 守护进程|
| valet use php@7.2 | 切换PHP 版本 |
| valet tld app | 切换顶级域名 |
