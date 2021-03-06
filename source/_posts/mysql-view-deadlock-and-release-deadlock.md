---
title: Mysql 查看死锁和解除死锁
date: 2020-07-19 16:07:48
tags: ["Mysql"]
categories: ["Mysql"]
---

前段时间遇到了一个Mysql 死锁相关的问题，整理一下。

<!-- more -->

> 问题描述：Mysql 的修改语句似乎都没有生效，同时使用Mysql GUI 工具编辑字段的值时会弹出异常。

![image.png](https://i.loli.net/2020/06/28/3dXRhKHQWMlearC.png)

### 什么是死锁
在解决Mysql 死锁的问题之前，还是先来了解一下什么是死锁。

死锁是指两个或两个以上的进程在执行过程中,因争夺资源而造成的一种互相等待的现象,若无外力作用,它们都将无法推进下去.此时称系统处于死锁状态或系统产生了死锁,这些永远在互相等的进程称为死锁进程。

### 死锁的表现
死锁的具体表现有两种：
1. Mysql 增改语句无法正常生效
2. 使用Mysql GUI 工具编辑字段的值时，会出现异常。

### 如何避免死锁
阻止死锁的途径就是避免满足死锁条件的情况发生，为此我们在开发的过程中需要遵循如下原则：

1.尽量避免并发的执行涉及到修改数据的语句。

2.要求每一个事务一次就将所有要使用到的数据全部加锁，否则就不允许执行。

3.预先规定一个加锁顺序，所有的事务都必须按照这个顺序对数据执行封锁。如不同的过程在事务内部对对象的更新执行顺序应尽量保证一致。

### 查看死锁
Mysql 查询是否存在锁表有多种方式，这里只介绍一种最常用的。

#### 1. 查看正在进行中的事务

```
SELECT * FROM information_schema.INNODB_TRX
```
可以看到 进程id为3175 的事务在锁住了，而另一个id为3173的事务正在执行，但是没有提交事务。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200719155146.png)

#### 2. 查看正在锁的事务

```
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200719163402.png)

#### 3. 查看等待锁的事务

```
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200719155801.png)

#### 4. 查询是否锁表

```
SHOW OPEN TABLES where In_use > 0;
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200719163422.png)

#### 5. 查看最近死锁的日志

```
show engine innodb status
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200719170929.png)

在发生死锁时，这几种方式都可以查询到和当前死锁相关的信息。

### 解除死锁
如果需要解除死锁，有一种最简单粗暴的方式，那就是找到进程id之后，直接干掉。

查看当前正在进行中的进程。
```
show processlist

// 也可以使用
SELECT * FROM information_schema.INNODB_TRX;
```
上面两个命令找出来的进程id 是同一个。

杀掉进程对应的进程 id
```
kill id
```

验证（kill后再看是否还有锁）

```
SHOW OPEN TABLES where In_use > 0;
```

### 参考链接
* [Mysql 查看表和解锁表](https://www.cnblogs.com/duanxz/p/4394641.html)
* [Mysql 死锁是什么？](https://blog.csdn.net/LJFPHP/article/details/80599352)