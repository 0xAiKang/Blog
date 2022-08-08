---
title: 轻量级开发环境 Valet 快速上手
date: 2022-08-08 18:31:34
tags: ["PHP", "Mac"]
categories: ["PHP", "Mac"]
---

> Valet 是为 Mac 提供的极简主义开发环境，没有 Vagrant ，也无需 `/etc/hosts` 文件，甚至可以使用本地隧道公开共享你的站点。\
Laravel Valet 会在你的 Mac 上将 Nginx 设置为随系统启动后台运行，然后使用 DnsMasq ， Valet 将所有的请求代理到 `*.test` 域名并指向本地安装的站点目录。\
换句话说，一个速度极快的 Laravel 开发环境仅仅需要占用 7MB 内存。 Valet 并不是想要替代 Vagrant 或者 Homestead，只是提供另外一种选择，更加灵活、方便、以及占用更小的内存。

<!-- more -->

## 安装 Valet
正式安装之前，首先更新一下 Homebrew。
```bash
brew update
```

然后安装各版本的 PHP：
```bash
brew install shivammathur/php/php@7.3
brew install shivammathur/php/php@7.2
brew install shivammathur/php/php@7.1
brew install shivammathur/php/php@7.0
brew install shivammathur/php/php@5.6
```

这里安装多个版本的目的是，为后续切换版本做准备。

安装完 PHP 之后，就可以使用 Composer 了，将 Valet 作为全局服务进行安装。

```bash
composer global require laravel/valet
```

安装完 Valet 之后，还不能直接使用，需要安装 Valet 所依赖的服务（DnsMasq）。

```bash
valet install
```

直到上一步完成，整个安装过程终于结束了。

为了验证是否安装成功，可以 ping 一下 `*.test` 的任意域名，如果可以 ping 通并看到 `127.0.0.1`，则表示服务正常。

![clipboard.png](inkdrop://file:mR3T8g8CN)

## 站点维护
使用 Valet 创建一个站点有两种方式：
* valet link
* valet park

上面两个命令都可以创建一个站点，`valet park` 算是 `valet link` 命令的升级版，可以一次创建 N 各站点。

### valet link

快速创建一个站点：
```bash
# cd /Users/boo/.config/valet/Sites/localhost 进入项目跟路径
$ valet link localhost
A [localhost] symbolic link has been created in [/Users/boo/.config/valet/Sites/localhost].
```

只需要一个命令，一个站点就创建好了：

![clipboard.png](inkdrop://file:pVFYCu5RW)

查看站点列表：
```bash
$ valet links
+-----------------------+-----+-----------------------------------+-----------------------------------------------------------------------+
| Site                  | SSL | URL                               | Path                                                                  |
+-----------------------+-----+-----------------------------------+-----------------------------------------------------------------------+
| localhost       |     | http://localhost.test       | /Users/boo/Projects/localhost
| +-----------------------+-----+-----------------------------------+-----------------------------------------------------------------------+
```

删除一个站点：
```bash
$ valet unlink localhost
```

### valet park

当某个目录下面有多个项目需要创建站点时，使用 `valet park` 尤为方便。

```bash
# cd /Users/boo/Sites 进入项目跟路径
$ valet park
This directory has been added to Valet's paths.
```

使用 `valet parked` 命令可以查看所有使用 park 添加的站点：

```bash
valet parked
+-----------+-----+-----------------------+----------------------------+
| Site      | SSL | URL                   | Path                       |
+-----------+-----+-----------------------+----------------------------+
| blog      |     | http://blog.test      | /Users/boo/Sites/blog      |
| localhost |     | http://localhost.test | /Users/boo/Sites/localhost |
| phpinfo   |     | http://phpinfo.test   | /Users/boo/Sites/phpinfo   |
+-----------+-----+-----------------------+----------------------------+
```

如果想把某个目录下面的所有站点都移除，可以使用 `valet forget` 命令，然后前提是这些站点都是使用 park 方式添加的。

`valet paths` 命令则是用来查看所有使用 `valet park` 添加站点的跟路径。

```bash
valet paths
[
    "/Users/boo/.config/valet/Sites",
    "/Users/boo/Sites",
]
```

## 切换版本
Valet 切换 PHP 版本非常方便，因为前面已经安装好了多版本的 PHP，所以可以直接使用下面的命令进行切换：

```bash
valet use php@7.3
```

## 其他命令
Valet 常用命令

|命令	|描述 |
| ------- | ------- |
|valet log | 从 Valet 的服务中查看日志 |
|valet restart	|重启 Valet 守护进程|
|valet start	|开启 Valet 守护进程|
|valet stop|	停止 Valet 守护进程|
|valet trust|	为 Brew 和 Valet 添加文件修改权限使 Valet 输入命令的时候不需要输入密码|
|valet uninstall	|完成卸载 Valet 守护进程|
| valet use php@7.2 | 切换PHP 版本 |
| valet tld app | 切换顶级域名 |

## 参考链接
* [Laravel Valet](https://laravel.com/docs/9.x/valet#installation)