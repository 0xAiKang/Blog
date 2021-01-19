---
title: PHP + xdebug 分析代码性能瓶颈
date: 2021-01-19 20:12:54
tags: ["PHP"]
categories: ["PHP"]
---

通常启用了`xdebug`插件，性能测试输出文件会伴随生成，通常是以`cachegrind.out.xxxx` 文件存在。

<!-- more -->

该文件可以通过第三方工具来进行代码性能分析。

但如果本地有多个项目/网站，所有的profile 都输出到一个文件中了，这样并不方便后面进行性能分析。

### 自定义profile 文件名称
可以通过配置`xdebug.profiler_output_name` 参数来设置输出文件名称，部分参数如下：

|符号|含义|配置样例|样例文件名|
|-|-|-|-|
|%c|当前工作目录的crc32校验值|cachegrind.out.%c|cachegrind.out.1258863198|
|%p|当前服务器进程的pid|cachegrind.out.%p|cachegrind.out.9685|
|%r|随机数|cachegrind.out.%r|cachegrind.out.072db0|
|%s|脚本文件名(注)|cachegrind.out.%s|cachegrind.out._home_httpd_html_test_xdebug_test_php|
|%t|Unix时间戳（秒）|cachegrind.out.%t|cachegrind.out.1179434742|
|%u|Unix时间戳（微秒）|cachegrind.out.%u|cachegrind.out.1179434749_642382|
|%H|$_SERVER['HTTP_HOST']|cachegrind.out.%H|cachegrind.out.localhost|
|%R|$_SERVER['REQUEST_URI']|cachegrind.out.%R|cachegrind.out._test_xdebug_test_php_var=1_var2|
|%S|session_id (来自$_COOKIE 如果设置了的话)|cachegrind.out.%S|cachegrind.out.c70c1ec2375af58f74b390bbdd2a679d|
|%%|%字符|cachegrind.out.%%|cachegrind.out.%%|

编辑`php.ini` 配置文件：
```
xdebug.profiler_output_name = cachegrind.out.%H
```
然后重启 php server。

在Mac 下，profile 文件存放于`/var/tmp/`目录中。

### 性能分析
在Mac 下，有MacCallGrind 和 qcachegrind 可以使用，不过前者是收费，直接通过Apple Store下载，后者是免费。需要手动安装。

安装graphviz，用来Call Graph功能：
```
$ brew install graphviz
```

安装 qcachegrind：
```
$ brew install qcachegrind
```

安装完成之后，就可以打开 `qcachegrind` 应用了，图形界面如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210119150021.png)

### 其他

不过需要注意，开启了`profile`文件输出之后，如果本地项目多的话，很容易占用磁盘大面积空间，下图是我半年左右没有清理的状态：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/%E7%A3%81%E7%9B%98.jpg)

可以使用命令进行清理：
```
$ sudo rm -fr /private/var/tmp/cachegrind.out.*
```

### 参考链接
* [使用xdebug对php进行profile的输出](https://xenojoshua.com/2011/05/xdebug-php-profile-output/)
* [php+xdebug+qcachegrind(mac)性能分析](https://segmentfault.com/a/1190000012395875)