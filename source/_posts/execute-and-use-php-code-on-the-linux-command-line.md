---
title: 在 Linux 命令行中执行和使用 PHP 代码
date: 2020-07-29 08:08:12
tags: ["PHP", "Shell"]
categories: ["PHP"]
---

众所周知，PHP是一门脚本语言，主要用于服务端（JavaScript 用于客户端）以通过HTTP 生成动态网页。

<!-- more -->

所以与其他脚本语言一样，可以直接在终端中不需要网页浏览器来运行PHP 代码。

## 获取安装信息
在安装完PHP 以及Nginx 之后，接下来我们通常需要做的是，在`/usr/local/var/www` (Mac 上的Nginx 工作目录)上创建一个内容为`<?php phpinfo(); ?>`，名为index.php的文件来测试PHP 是否安装正确。

执行以下命令即可：
```
# echo '<?php phpinfo(); ?>' > /usr/local/var/www/index.php
```

然后，使用浏览器访问`http://127.0.0.1/index.php`，不出意外可以看到：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200930151004.png)

> 如何在终端中直接查看该信息？
```
# php -f /usr/local/var/www/index.php | less
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200930151053.png)

如果你觉得上面这种方式太麻烦了，那么还有一种更简便的方式可以达到同样的效果。
```
# php -r 'php phpinfo();' | less
```
### 交互模式
有时候我们会遇到这样一种情况，想测试一小段代码，看看其运行结果，但是又不想重新创建一个文件，太麻烦了。

如果这个时候有一个地方可以直接运行这段代码且输出结果，那该多好啊。

PHP 为我们提供了两种交互模式，前者是自动的，后者是手动的。
1. Interactive shell
2. Interactive mode enabled

两种模式都是使用 `php -a` 命令进入。

#### Interactive shell

使用这个交互式shell，你可以直接在命令行窗口里输入PHP并直接获得输出结果。
```
$ php -a
Interactive shell

php >echo "Hello PHP";
Hello PHP
php > echo 10+90;
100
```
回车即可查看输出内容。

#### Interactive mode enabled

```
$ php -a
Interactive mode enabled

php >echo "Hello PHP";
```
如果出现的是这个模式，说明你的PHP并不支持交互式shell，

不过不用担心，这个模式同样也可以执行PHP 代码，只是代码的执行方式有些区别。

输入了所有PHP代码后，输入`Ctrl-Z`（windows里），或输入`Ctrl-D`（linux里），你输入的所有代码将会一次执行完成并输出结果。

输入`exit`或者`⌃ + c` 退出交互模式。

### PHP 脚本
在终端中可以把PHP 脚本作为Shell 脚本来运行。

首先你需要创建一个PHP 脚本文件：
```
# echo -e '#!/usr/bin/php\n<?php phpinfo();?>' > phpscript.php
```
`-e` 表示激活转义字符。

注意，这个脚本文件中的第一行`#!/usr/bin/php`，就像是Shell 脚本中的`#!/bin/bash`。目的是告诉Linux 命令行使用PHP 解析器来解析该文件。

运行该脚本：
```
# chmod +x phpscript.php  // 使脚本具有执行权限
# ./phpscript.php   //执行脚本
```

### PHP 服务
PHP 有内置一个WebServer，可以很方便快速的搭建一个PHP 服务。

```
$ php -t /project to path -S localhost:port
```
然后通过浏览器访问`localhost:port` 就可以了。

### 总结
* `php -a`：进入交互模式
* `php -f`：解析和执行文件
* `php -h`：获取帮助
* `php -i`：查看PHP 信息和配置
* `php -m`：显示已经安装的模块
* `php -r`：运行PHP代码不使用脚本标签'<?..?>'
* `php -v`：查看PHP 版本
* `php -ini`：查看加载配置文件（php.ini、conf.d）
* `php -i | grep configure`：查看静态编译模块
* `php --ri swoole`：查看指定模块的配置
* `locate php.ini`：查询本地配置文件
* `time php script.php`：查看程序的执行时间

### 参考链接
* [在 Linux 命令行中执行和使用 PHP 代码](https://linux.cn/article-5906-1.html)
* [12 个 Linux 终端中有用的 PHP 命令行用法](https://www.tecmint.com/execute-php-codes-functions-in-linux-commandline/)