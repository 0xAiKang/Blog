---
title: 如何在 Mac 上使用 Docker 运行 PHP
date: 2023-05-18 14:07:56
tags: ["PHP"]
categories: ["PHP"]
---

在之前的一篇笔记中，介绍了，[如何为 PHP 编写 Dockerfile](https://www.0x2beace.com/how-to-write-dockerfile-for-php/)。

编写好 Dockerfile 之后，就可以构建成镜像了，推送到自己的 Dockerhub，之后想要使用就很方便了。

这篇笔记，来介绍，如何在 Mac 上使用 Docker 运行 PHP。

除了 PHP 是使用容器，Mysql、Nginx、Redis 等服务都是跑在本地。

<!-- more -->

## 创建 PHP 容器
为了保证，每次重启容器，IP 不会发生变化，在创建容器的时候，需要固定容器 IP。

因此，可以单独创建一个 network，所有的 PHP 容器都放在这个网络下。

创建 network：
```bash
$ docker network create --subnet=172.22.0.0/24 bridge0
```

创建 PHP 容器：
```bash
$ docker run -d --name php-7.1.33 --net bridge0 --ip 172.22.0.5 -p 7133:9000 -v ~/Projects:/var/www hoooliday/php:7.1.33
$ docker run -d --name php-7.2.33 --net bridge0 --ip 172.22.0.2 -p 7233:9000 -v ~/Projects:/var/www hoooliday/php:7.2.33
$ docker run -d --name php-7.3.33 --net bridge0 --ip 172.22.0.3 -p 7333:9000 -v ~/Projects:/var/www hoooliday/php:7.3.33
$ docker run -d --name php-7.4.33 --net bridge0 --ip 172.22.0.2 -p 7433:9000 -v ~/Projects:/var/www hoooliday/php:7.4.33
$ docker run -d --name php-8.0.28 --net bridge0 --ip 172.22.0.6 -p 8028:9000 -v ~/Projects:/var/www hoooliday/php:8.0.28
$ docker run -d --name php-8.1.14 --net bridge0 --ip 172.22.0.7 -p 8114:9000 -v ~/Projects:/var/www hoooliday/php:8.1.14
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230510170552.png)

不同版本的 PHP，对应监听本地不同的端口，需要切换版本时，更改转发端口即可。

## Nginx 配置

在 Mac 上使用 PHP 容器 + Nginx 部署网站时，Nginx 通常充当反向代理服务器，将请求转发到 PHP 容器，而静态资源（例如 CSS、JavaScript、图片等）通常存储在宿主机上，而不在 PHP 容器中。

因此，在 Nginx 配置文件中，对于动态资源，需要配置容器的绝对路径; 对于静态资源这一块，需要配置宿主机的绝对路径。

示例配置：
```nginx
server {
        # 宿主机对外提供服务端口
        listen 6350;

        error_page 404 /404.html;
        error_page 500 502 503 504 /50x.html;

        location ~ \.php {
                # 项目在容器中的所在路径
                root /var/www/Work/easycommerce/oceanus/public;
                # 转发到对应容器
                fastcgi_pass 127.0.0.1:7433;
                include   fastcgi.conf;
                set $path_info "";
                set $fastcgi_script_name_new $fastcgi_script_name;

              if ($fastcgi_script_name ~*   "^(.+\.php)(/.+)$"  ) {
                        set $fastcgi_script_name_new $1;
                        set $path_info $2;
                }
                        fastcgi_param   SCRIPT_FILENAME   $document_root$fastcgi_script_name_new;
                        fastcgi_param   SCRIPT_NAME   $fastcgi_script_name_new;
                        fastcgi_param   PATH_INFO $path_info;
        }

        location / {
                # 项目在宿主机中的所在路径
                root ~/Projects/Work/easycommerce/oceanus/public;
		index index.php index.html index.htm;
                if (!-e  $request_filename){
                        rewrite ^(.*)$ /index.php$1 last;
                }
        }
    }
```

## 访问宿主机网络

使用Bridge 网络模式，在容器中，需要访问宿主机网络，这里提供一个最简单的方案。

Docker for Mac 提供了一个指向宿主机的域名 `host.docker.internal`，在需要访问宿主机服务时使用此域名即可，如果是旧版本，则需要使用较旧的别名 `docker.for.mac.host.internal`。

在容器中，ping 宿主机域名，可以看到，是能正常访问的：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230510165739.png)

将项目中需要用到宿主机IP 的位置，换成 `host.docker.internal` 即可正常访问到宿主机服务（Mysql、Redis）。