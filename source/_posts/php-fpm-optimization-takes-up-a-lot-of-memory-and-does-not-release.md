---
title: PHP-FPM 优化——占用内存大不释放
date: 2020-10-18 15:56:45
tags: ["PHP", "PHP-FPM"]
categories: ["PHP"]
---

在传统的 LNMP 架构中，如果Web 应用部分，突然变得特别卡，通常都是内存耗尽导致。

<!-- more -->

这里说的内存，指的是物理运行内存，而不是虚拟内存（Swap）。

LNMP架构中PHP是运行在FastCGI模式下，按照官方的说法，php-cgi会在每个请求结束的时候会回收脚本使用的全部内存，但是并不会释放给操作系统，而是继续持有以应对下一次PHP请求。而php-fpm是 FastCGI进程管理器，用于控制php的内存和进程等。

所以，解决的办法就是通过php-fpm 优化总的进程数和单个进程占用的内存，从而解决php-fpm 进程占用内存大和不释放内存的问题。

### 查看当前占用情况

如果发现Web 应用出现严重卡顿，请求超时等问题，首先检查一下内存的占用情况。常用的命令有：Top、Glances、Free 等。

使用Glances 或者 Top 命令查看进程，然后按下按键 M，可以查看主机当前的内存占用情况，按照占用内存由多到少排序。

也可以使用以下命令查看当前 php-fpm 总进程数：
```
ps -ylC php-fpm --sort:rss
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201018104932.png)

其中 rss 就是内存占用情况。

查看当前php-fpm 进程的内存占用情况及启动时间：
```
ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid'|grep www|sort -nrk5
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201018105106.png)

可以看到无论哪一种方式，结果都是一样的。

查看当前php-fpm进程平均占用内存情况：
```
ps --no-headers -o "rss,cmd" -C php-fpm | awk '{ sum+=$1 } END { printf ("%d%s\n", sum/NR/1024,"M") }'
// 22M
```

### 优化配置
解决上面那个问题的核心就是 php-fpm 配置中的 `max_requests`。

即当一个 PHP-CGI 进程处理的请求数累积到 max_requests 个后，自动重启该进程，这样达到了释放内存的目的了。

一般来说一个php-fpm 进程占用的内存为30M-40M，所以根据自身实际情况作判断，有以下两种情况：
1. 实际结果是大于 30M - 40M，那么需要让 php-fpm “早一些“释放内存，`max_requests` 的数值改小一些。
2. 实际结果是小于 30M - 40M，则可以让 php-fpm “晚一些“释放内存，`max_requests`的数值改大一些。

### 参考链接
* [Linux的php-fpm优化心得-php-fpm进程占用内存大和不释放内存问题](https://wzfou.com/php-fpm/)