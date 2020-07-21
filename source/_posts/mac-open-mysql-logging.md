---
title: Mac 开启 Mysql 日志记录
date: 2020-07-19 17:24:43
tags: ["Mysql", "Mac"]
categories: ["Mysql"]
---

有时候可能会想在本地开启Mysql 的日志记录，看看具体都执行了哪些SQL，其实非常简单。

<!-- more -->

1. 进入Mysql 命令行

```
mysql -hlocalhost -uroot -p
```

2. 全局开启普通日志记录

```
set global general_log=on;
```

3. 查看Mysql 日志文件所在目录

```
show variables like 'general_log_file';
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200719172134.png)

4. 实时查看日志记录

```
tail -f /your_mysql_log_file_path
```
