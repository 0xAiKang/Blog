---
title: php-fpm configuration initialization
date: 2020-09-06 11:57:11
tags: ["PHP", "PHP-FPM"]
categories: ["PHP"]
---

`php-fpm`（FastCGI Process Manger）是一个PHP FastCGI 管理器，专门和Nginx 的 `ngx_fastcgi_modul`模块对接，用来处理动态请求。

<!-- more -->

## 初始化
当安装了PHP 之后，可以从以下三个方向来对默认配置进行修改，以达到优化的效果。

### 1. 核心配置文件
核心配置文件其实就是 `php.ini`，该配置文件的作用通常是用来启用或禁用第三方模块，及修改PHP 时区等。

```
# vim /usr/local/etc/php/php.ini

date.timezone = Asia/Shanghai
```

### 2. 全局配置文件
全局配置文件`php-fpm.conf`，通常用来配置一些辅助性功能。

```
# vim /usr/local/etc/php-fpm.conf

error_log = /var/log/php-fpm/error.log
log_level = notice
;process_max = 0
deamonize = yes
```

参数解析：
* `error_log`：错误日志路径
* `log_level`：日志级别，默认为notice
  * `alert`：必须立即处理
  * `error`：错误情况
  * `warning`：警告情况
  * `notice`：一般重要信息
  * `debug`：调试信息
* `process_max`：控制最大子进程数的全局变量，不建议设置具体数量，因为会限制扩展配置。
* `daemonize`：是否开启守护进程，默认为yes

通常不会在`php-fpm.conf`中设定 `process_max`，因为会限制`www.conf`中的配置。

### 3. 扩展配置文件
扩展配置文件`www.conf`通常是与`php-fpm`服务相关的配置，大部分优化都是需要更改这个配置文件。

```
# vim /usr/local/etc/php-fpm.d/www.conf

listen = 127.0.0.1:9000
slowlog = /var/log/php-fpm/www-slow.log

# 这里按照10G 的空闲内存去设定
pm = dynamic
pm.start_servers = 16
pm.max_children = 256
pm.min_spare_servers = 16
pm.max_spare_servers = 32
pm.max_requests = 1000
```

参数解析：
* `listen`：有两种方式可以进行通讯。
  * `socket`：`unix:/run/php/php7.3-fpm.sock`
  * `http`：`127.0.0.1:9000` 因为`php-fpm`与`ngx_fastcgi_modul`的通讯方式是 9000端口，所以默认是 `127.0.0.1:9000`
* `slowlog`：慢查询日志路径
* `pm`：进程管理方式
  * `static`：静态模式。始终保持固定数量的子进程数，配合最大子进程数一起使用，这个方式很不灵活，通常不是默认。
    * `pm.max_children`：最大子进程数。
  * `dynamic`：动态模式。按照固定的最小子进程数启动，同时用最大子进程数去限制。
    * `pm.start_servers`：默认开启的进程数
    * `pm.min_spare_servers`：最小空闲的进程数
    * `pm.max_spare_servers`：最大空闲的进程数
    * `pm.max_children`：最大子进程数
    * `pm.max_requests`：每个进程能响应的请求数量，达到此限制之后，该PHP 进程就会被自动释放掉。
  * `nodaemonize`：每个进程在闲置一定时候后就会被杀掉。
    * `pm.max_children`：最大子进程数
    * `pm.process_idle_timeout`：在多少秒之后，一个空闲的进程将会被杀死

注意：`max_children` 是 PHPFPM Pool 最大的子进程数，它的数值取决于服务器实际空闲内存。假设你有一台10G 运行内存的服务器，我们知道一个空闲的PHP 进程占用的是 1M 内存，而一个正在处理请求的PHP 进程 大概会占用`10M-40M`内存，这里按照每个PHP 请求占用 40M 内存，那么`max_children = 10*1024M/40M = 256`，所以这个值得根据实际环境而设定。

以上就是`php-fpm` 初始化配置的核心部分了。