---
title: Docker 数据挂载
date: 2020-10-07 20:24:01
tags: ["Docker"]
categories: ["Docker"]
---

### 数据挂载
数据挂载在Docker 中还是挺重要的一部分，因为有多种方式，而不同的方式所对应的处理数据的逻辑也不一样。

1. Volumes：Docker 管理宿主机文件系统的一部分（/var/lib/docker/volumes）。
2. Bind Mounts：将宿主机上的任意位置的文件或目录挂载到容器中。
3. tmpfs：挂载存储在主机系统的内存中，而不会写入主机的文件系统。如果不系统将数据持久存储在任何位置，可以使用tmpfs，同时避免写入容器可写层提高性能。

这里主要介绍前两者，后者使用的并不多。注意第一种和第二种是存在区别的，前者是使用的数据卷进行挂载，而后者则是直接使用的宿主机上的文件或者目录挂载到容器中。

众所周知，将容器删除之后，容器内所有的改动将不复存在。

挂载数据卷通常是最常用且最好的方式，这种方式会将容器中的数据持久化在宿主机中，这样做的好处就是当容器被删除或者无法正常启动时，数据仍是完整的。

挂载数据卷有两种方式：
1. 使用`--mount`
2. 使用`-v`

前者是新版本的方式，后者是老版本的方式，其效果都是一样的。

#### Volumes
创建一个数据卷：
```
docker volume create <volume name>
```

列出数据卷列表：
```
docker volume ls
```

列出数据卷的详情信息：
```
docker volume inspect <volume name>
```

删除数据卷：
```
docker volume rm <volume name>
```

用数据卷创建一个容器：
```
# 新版本
docker run -d -it \
--name=nginx --mount src=<volume name>,dst=/usr/share/nginx/html \
nginx

# 老版本
docker run -d -it \
--name=nginx -v <volume name>:/usr/share/nginx/html \
nginx
```

需要注意的是：
1. 如果没有指定数据卷，则会自动创建

#### Bind Mounts
使用bind mounts 创建一个容器：
```
# 新版本
docker run -d -it \
--name nginx \
-p 8080:80 \
--mounts type=bind,src=/var/www,dst=/usr/share/nginx/html \
nginx

# 老版本
docker run -d -it \
--name nginx \
-p 8080:80 \
-v /var/www:/usr/share/nginx/html \
nginx
```

需要注意的是： 
1. 如果源文件/目录没有存在，docker 不会自动创建，而会自动抛出一个错误。
2. 如果挂载目标在容器中是非空目录，则该目录现有内容将被隐藏。