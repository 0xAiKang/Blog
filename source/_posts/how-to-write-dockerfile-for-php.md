---
title: 如何为PHP编写 Dockerfile
date: 2023-04-04 18:24:39
tags: ["Docker"]
categories: ["Docker"]
---

[PHP 官方镜像](https://hub.docker.com/_/php)有很多不同的版本，适用于不同的应用场景和需求。

<!-- more -->

## 如何选择基础镜像

主要分为三个分支：
1. cli：开启了 CLI
2. fpm：开启了 CLI 并支持 Web 服务
3. zts：开启了线程安装的版本

以 `php-7.4` 为例，常见的版本有 `7.4`、`7.4.33-zts`、`7.4.33-cli`、`7.4.33-fpm`、`7.4-apache`。

下面一一介绍：
* `7.4`：是一个基于 PHP 7.4 版本的镜像，包含了 PHP 的基本环境和常用扩展，适用于大多数 PHP 应用程序的部署。该镜像支持 CLI 和 CGI 模式，但不支持 FPM 模式。
* `7.4.33-zts`：是一个基于 PHP 7.4.33 版本的镜像，支持多线程（ZTS）模式，适用于需要在多线程环境下运行 PHP 应用程序的场景。该镜像支持 CLI 模式。
* `7.4.33-cli`：是一个基于 PHP 7.4.33 版本的 CLI 镜像，适用于需要在命令行下运行 PHP 应用程序的场景。该镜像不包含 Web 服务器，只包含 PHP CLI 环境和常用扩展。
* `7.4.33-fpm`：是一个基于 PHP 7.4.33 版本的 FPM 镜像，适用于需要使用 PHP-FPM的场景。在使用该镜像时，需要将 PHP 代码与一个 Web 服务器（如 Nginx 或 Apache）结合使用，以提供 Web 服务。
* `7.4-apache`：是一个基于 PHP 7.4 版本的 Apache 镜像，适用于需要使用 Apache 作为 Web 服务器的场景。该镜像已经预装了 Apache 和 PHP，可以直接用于 Web 应用程序的部署。在使用该镜像时，需要将 PHP 代码放置在 Apache 的 Web 目录中，以便 Apache 可以正确地解释和执行 PHP 代码。

### cli

`7.4`、`7.4.33-cli` 这些镜像都支持 CLI 模型，而不支持 FPM 模式。

因此通常用于执行一些脚本文件。
```bash
docker run --rm --name php-74cli \
  -v /www/wwwroot:/www/wwwroot \
  php:7.4-cli \
  php /www/wwwroot/script.php
```

或者启动一个交互式的命令行：
```bash
docker run -it --name php-74cli \
-v /www/wwwroot:/www/wwwroot \
php:7.4-cli
```

参数说明：
* `--rm`：不保留容器，运行完毕，自动删除
* `-it`：开启一个交互式操作的Shell
* `-v`：挂载宿主机的目录
* `php /www/wwwroot/script.php`：容器启动之后，执行脚本

cli 的镜像是不能已后台运行的方式启动的，因为本身只是启动一个交互式的命令行。

### fpm

`7.4.33-fpm`、`7.4.33-apache` 这些镜像是支持 Web 服务的，

```bash
docker run -d --name php-74fpm \
-p 9000:9000 \
-v /www/wwwroot:/www/wwwroot \
php:7.4-fpm
```

参数说明：
* `-d`：让容器在后台运行
* `-p`：端口号映射，将本机 9000 端口与容器的 9000 端口做映射

### alpine
alpine 版本的镜像，是基于 Alpine Linux 的轻量级镜像，上面的每个版本都有对应的 alpine 版本：
* `7.4-alpine`
* `7.4.33-zts-alpine`
* `7.4.33-cli-alpine`
* `7.4.33-fpm-alpine`
* `7.4-apache-alpine`

如果要编写一个生产环境可用的镜像，最好是基于 `fpm-alpine` 进行构建，这样构建出来的镜像功能是最完整的，支持 CLI、CGI、FPM 模式。

## 编写 Dockerfile
接下来基于 `7.4.33-fpm-alpine` 这个镜像，来编写一个 Dockerfile。

可以直接根据官方的镜像，构建应用，为什么还需要自己编写 Dockerfile？

其实是很有必要的，这是因为基于官方的镜像进行构建，还有很多东西是不全的，就比如 composer 就没有安装，再就是许多扩展。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230307180655.png)
因此，自己编写一个 Dockerfile，定制容器其实是很有必要的。

首先需要确定，还需要哪些扩展：
* bcmath
* bz2
* calendar
* dba
* exif
* gd
* opcache
* gettext
* intl
* imap
* mongodb
* mysqli
* pcntl
* pdo_mysql
* pdo_pgsql
* pgsql
* redis
* soap
* swoole
* sysvmsg
* sysvsem
* sysvshm
* xsl
* tidy
* zip

以及 composer。

完整 Dockefile 如下：
```Dockerfile
# Dockerfile
# docker build -f 7.4.33.Dockerfile . -t php:7.4.33

# 使用 7.4.33-fpm-alpine3.16 作为基础镜像
FROM php:7.4.33-fpm-alpine3.16 as builder
LABEL MAINTAINER="0xAiKang <aikangtongxue@gmail.com>"
LABEL description="PHP 镜像，版本为 7.4，支持 CLI、CGI、FPM 模式"

# 更新下载源
RUN echo 'https://mirrors.aliyun.com/alpine/v3.16/main/' > /etc/apk/repositories && \
    echo 'https://mirrors.aliyun.com/alpine/v3.16/community/' >> /etc/apk/repositories

# 安装依赖库和工具
RUN apk add --no-cache --virtual .build-deps \
    autoconf \
    gcc \
    g++ \
    make \
    icu-dev \
    libzip-dev \
    libxml2-dev \
    oniguruma-dev \
    bzip2-dev \
    gettext-dev \
    libmcrypt-dev \
    libwebp-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    libxslt-dev \
    openssl-dev \
    krb5-dev \
    unixodbc-dev \
    postgresql-dev \
    openldap-dev \
    tidyhtml-dev \
    freetds-dev \
    zlib-dev \
    c-ares-dev \
    curl-dev \
    imap-dev \
    && apk add --no-cache \
    git \
    curl \
    icu \
    libintl \
    libzip \
    libxml2 \
    libxslt \
    libwebp \
    freetype \
    libjpeg-turbo \
    libpng \
    tidyhtml-libs \
    unixodbc \
    postgresql-libs \
    openldap \
    gettext

# 安装 PHP 扩展
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-configure intl \
    && docker-php-ext-configure imap --with-imap --with-imap-ssl \
    && docker-php-ext-install \
    imap \
    bcmath \
    bz2 \
    calendar \
    dba \
    exif \
    gd \
    opcache \
    gettext \
    intl \
    mysqli \
    pcntl \
    pdo_mysql \
    pdo_pgsql \
    pgsql \
    soap \
    sysvmsg \
    sysvsem \
    sysvshm \
    xsl \
    tidy \
    zip

# 通过 PECL 安装扩展
RUN pecl install mongodb redis \
    && docker-php-ext-enable mongodb redis \
    && pecl install -D 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole-4.8.13 \
    && docker-php-ext-enable swoole

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 切换到一个新的阶段，以减少最终镜像大小
FROM php:7.4.33-fpm-alpine3.16

# 复制已安装的扩展和配置
COPY --from=builder /usr/local/etc/php /usr/local/etc/php
COPY --from=builder /usr/local/lib/php /usr/local/lib/php

# 安装运行时依赖
RUN apk add --no-cache \
    icu \
    libintl \
    libzip \
    libxml2 \
    libxslt \
    libwebp \
    freetype \
    libjpeg-turbo \
    libpng \
    tidyhtml-libs \
    unixodbc \
    postgresql-libs \
    openldap \
    gettext \
    imap-dev \
    c-ares-dev

# 复制已安装的 Composer
COPY --from=builder /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www

# 清理缓存和无关文件
RUN rm -rf /var/cache/apk/*

# 暴露 FPM 服务端口
EXPOSE 9000
```


需要注意：
* 为了构建一个尽可能小的 PHP 容器，可以使用多阶段构建来减小镜像大小。首先在 builder 阶段安装了所有的依赖和扩展，然后将安装好的扩展和配置复制到一个新的基础镜像中，从而减小了最终镜像的大小。
* 如果宿主机所在环境，没有科学上网的条件，需要将 alpine 的镜像下载源替换为国内阿里云镜像站，否则构建速度会非常慢，且很可能会构建失败。
* 制作镜像时，要注意不要留下太多运行时的文件，保证最小镜像体积
* 使用 pecl 安装swoole，默认是最新版本的，而最新版本需要 PHP 版本大于 8.0，因此这里需要安装指定版本

经过测试，该 Dockerfile 是可以正常构建出 PHP-7.4.33 版本的镜像，大小为 160 M。

## 推送至镜像仓库

构建镜像：
```bash
$ docker build -f php-7.4.33.Dockerfile . -t php:7.4.33
```

创建 Tag：
```bash
$ docker tag php:7.4.33 hoooliday/php:7.4.33
```

发布镜像：
```bash
$ docker push hoooliday/php:7.4.33
```

## 其他问题

### 如何在镜像中安装扩展

使用 `docker-php-ext-install` 命令，可以安装除 `mongodb`、`redis`、`swoole`、`xdebug`外的扩展。
```bash
$ docker-php-ext-install bcmatch
```

使用 `pecl install` 安装 `mongodb`、`redis`、`swoole`、`xdebug`等扩展：
```bash
$ pecl install xdebug-3.0.3 && docker-php-ext-enable xdebug
```

使用 `docker-php-ext-enable`命令启用扩展。

手动安装扩展时，可能会缺少必要的构建工具和库，可以使用以下命令安装：
```bash
$ apk add --no-cache autoconf build-base 
```

### 如何查看扩展包信息

```php
$ php -r "print phpinfo();" | grep ".ini"

# 或者
$ php -i | grep ini
```

### 无法加载动态库

如果某个扩展包，提示无法加载动态库，这时可根据提示，安装缺少的依赖包：

```bash
$ apk update && apk add libc-client-dev
```

## 参考链接
* [Docker PHP 官方镜像](https://github.com/docker-library/php/tree/b9f17156020c3aef71df681b27684533529347a7)