---
title: 记一次升级 PHP 引发的一些思考
date: 2021-03-31 23:27:38
tags: ["PHP", "一些思考"]
categories: ["PHP", "一些思考"]
---

因为工作原因，今天将本地开发环境的PHP 升级到7.4 了，此前一直使用7.3。

中间遇到了一些小问题，总体还算顺利，在此记录一下。

<!-- more -->

### 背景说明
我并没有使用集成的开发环境，而是单独安装所需的`5.6`、`7.0`、`7.1`、`7.2`、`7.3` 版本，所以升级`7.4` 也很简单，直接使用`brew` 安装即可：

```
$ brew install php@7.4
```

但是这会带来一个新的问题：之前通过源码编译安装过的扩展，还需要再安装一次。

你可能会问，为什么还需要再安装一次呢？直接把`php.ini` 中的开启扩展配拷贝过去不就可以了吗？

我们来试试这样做会发生什么？

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210331215353.png)

可以看到 PHP 并没有正常加载该扩展，这是为啥呢？

要回答这个问题，首先我们需要搞清楚，源码编译安装是怎么回事。

当我们执行`phpize` 命令后，会根据当前系统信息（PHP 版本）生成对应版本的扩展文件。

所以PHP7.3 编译生成的扩展自然就不能直接拿到PHP 7.4 中去使用了。

### xdebug

另外想说一下Xdebug ，它是我一直在使用的一个调试扩展，非常强大。

在PHP 升起到7.4 之后，我一并安装了最新版的Xdebug（3.x），此前我一直使用 2.x 版本的，因为版本跨度比较大，刚开始问题挺多的，断点总是进不去。

起初我认为是新旧配置不兼容，挺多参数名称发生了变化，（具体可以看[这里](https://xdebug.org/docs/upgrade_guide/en)），当我把配置全部切换成适应新版本，还是进不去。

后来阴差阳错升级了PHPStorm，结果就能调试了...（升级之前的版本是 2020.1）

适应`xdebug 3.x`的配置如下：
```
[XDebug]
zend_extension=/usr/local/lib/php/pecl/20190902/xdebug.so
xdebug.mode = debug
xdebug.client_host = 127.0.0.1
xdebug.client_port = 9003
xdebug.idekey=PHPSTORM
```

只是到最后我也没整明白到底是啥原因导致。