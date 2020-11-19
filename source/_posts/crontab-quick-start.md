---
title: Crontab 快速上手
date: 2020-10-27 23:22:29
tags: ["Linux", "Crontab"]
categories: ["Linux"]
---

[Crontab](https://zh.wikipedia.org/zh-hans/Cron) 是Unix 系统中基于时间的任务管理工具。

<!-- more -->

这个命令与传统的 Unix 命令不一样，下面会一一介绍其规则及其用法。

## crontab 还是 cron
[crontab](https://baike.baidu.com/item/crontab) 还是 [cron](https://baike.baidu.com/item/cron)？初次接触 crontab 的同学可能会被这两个词给绕晕。

其实可以这样来理解：`crontab`就是 `cron`服务的命令行工具，而`cron`则是背后处理`crontab`投递任务的服务。

### 文件格式
crontab 命令是以固定的时间格式来使用的，

|表示意义|分钟|小时|日期|月份|周|命令|
|-|-|-|-|-|-|-|
|范围|0～59（*）|0～23（*）|1～31（*）|1～12（*）|0～7（*）|需要执行的命令|

另外还有一些特殊字符具有特殊含义：
* `*` 表示任何时刻都接收。举个栗子：`* 12 * * *` 表示不论何月、何日的星期几的十二点都执行指定命令。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201027163458.png)

### 常用实例
每分钟执行一次：
```
*/1 * * * * 或者 * * * * *
```

每五分钟执行一次：
```
*/5 * * * *
```

每小时执行一次：
```
0 * * * *  或者 0 */1 * * *
```

每天执行一次：
```
0 0 * *  *
```

每周执行一次：
```
0 0 * *  0
```

每月执行一次：
```
0 0 1 *  0
```

### 如何使用
初次接触`crontab` 命令时，我也比较纳闷，这个命令倒底是如何使用的？

使用 crontab 有两种方式：
1. crontab -e：直接接受标准输入（键盘）上键入的命令，并将它们载入crontab。
2. crontab file：将file 作为crontab 的任务列表文件并载入crontab

第一种方式没什么好说的，直接在终端添加 crontab 任务就行了，下面简单说一下第二种（其实两者的核心都是一样的）。

#### 创建crontab 文件
首先创建一个文件，该文件的内容以**功能描述**、**执行时间**、**执行任务** 这几部分组成。

其中，前两者并不是一定需要，只是为了方便自己日后或其他人能快速知道这个任务具体是做什么的，`#` 表示注释。

示例，创建一个名称为`script_cron` 的crontab 文件：
```
# 每分钟执行一次 script.php 脚本
* * * * * /usr/bin/php ~/script.php
```

#### 运行crontab

为了提交刚刚创建的crontab 文件，可以把这个新创建的文件名称作为`crontab`命令的参数：
```
$ crontab script_cron
```

#### 列出cron 服务
使用`-l` 参数列出crontab文件：
```
$ crontab -l
# 每分钟执行一次 script.php 脚本
* * * * * /usr/bin/php ~/script.php
```

#### 编辑cron 服务

```
$ crontab -e
```

#### 删除cron 服务

```
$ crontab -r
```

### 常见问题

#### crontab 没有立即生效
新创建的cron 任务，不会马上执行，至少要过两分钟才执行。

如果希望能马上执行，可以重启 crontab 。


```
// Ubuntu：
$ service cron restart    

// Centos
$ service crond restart
```

#### crontab 压根没执行
有时候会遇到直接在命令行中可以执行任务，但是定时任务却怎么都不执行，

这时首先需要确认 cron 服务是否正常：
```
// Ubuntu：
$ service cron status    

// Centos
$ service crond status
```

然后确认需要执行的任务是否包含路径，如果包含请使用全局路径。

最后重启 cron 服务，通常到这里就已经可以正常执行了，如果还不行，尝试引入环境变量：

```
0 * * * * . /etc/profile; /usr/bin/php /var/www/script.php
```

#### crontab 无权限执行
需要注意的是crontab 任务的调度，只有 root 和任务所有者拥有权限。

如果想要编辑/查看/删除其他用户的任务，可以使用以下命令：

```
$ crontab -u <username> <选项>
```

常用选项：
`-e`：编辑任务
`-l`：查看任务
`-r`：删除任务

#### 查看 crontab 任务执行情况

当定时任务在指定时间执行时，会同步输出类似日志：
```
$ tail -f /var/log/syslog
Nov 19 12:47:01 gigabit CRON[14521]: (root) CMD (/usr/bin/php /var/www/script.php)
```
此时就可以肯定任务调度正常。


### 参考链接
* [crontab用法与实例](https://www.linuxprobe.com/how-to-crontab.html)
* [19. crontab 定时任务](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html)
* [Linux Crontab命令定时任务基本语法与操作教程-VPS/服务器自动化](https://wzfou.com/crontab/#ftoc-heading-2)