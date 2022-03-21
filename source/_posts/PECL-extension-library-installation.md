---
title: PECL 扩展库安装
date: 2022-02-12 14:44:53
tags: ["PHP"]
categories: ["PHP"]
---

# PECL 扩展库安装

通常安装 PHP 扩展，有两种方式：
1. 源码编译安装
2. 通过  pecl 命令一键安装

<!-- more -->

## 源码编译安装

源码编译安装的好处是更灵活，如果有进一步的需求，可以根据具体需要和版本，调整相关编译参数。

通常步骤如下：

1. 下载源码

* [RECL](https://pecl.php.net)
* [GitHub](https://github.com/)

2. 从源码编译安装

```bash
phpize && \
./configure && \
make && sudo make install
```

* `phpize`：来生成编译检测脚本
* `./configure`：来做编译配置检测
* `make`：进行编译
* `make install`：进行安装

如果需要指定配置文件位置，可以增加`--with-php-config=/php-config-path` 参数。

3. 启用扩展

编译安装到系统成功后，需要在 `php.ini` 中加入一行 `extension=extension.so` 来启用对应扩展。

```ini
extension = extension.so
  
# 可以是绝对路径
extension = "/php-config-path/extension.so"
```

## PECL 命令行安装

PECL 发布时间通常指晚于 GitHub 发布时间。

对于已经收录到 PHP 官方扩展库的扩展，除了手动下载编译外，还可以通过 PHP 官方提供的 `pecl` 命令，一键下载安装。

1. 更新 pecl 库

```shell
sudo pecl channel-update pecl.php.net
```

2. 使用 pecl 一键安装扩展

```shell
sudo pecl install mongodb
```

3. 查看当前运行 php 配置文件位置

```shell
php --ini
```

4. 启用扩展

```ini
extension = extension.so
  
# 可以是绝对路径
extension = "/php-config-path/extension.so"
```

5. 重启服务

```bassh
sudo service php7.4-fpm restart
php -m | grep mongodb
php --ri mongodb
```