---
title: 如何查看 Linux 默认时区
date: 2020-08-03 22:03:08
tags: ["Linux"]
categories: ["Linux"]
---

最近遇到一个跟服务器时区相关的问题，没准备充分，当问题真正来临时，很懵。

特别是在生产环境中，系统时区是特别重要的存在，很多应用在默认情况下，都是取的系统时区，如果时区处理不得当的话，可能会造成不必要的困扰。

<!-- more -->

# 时区的概念
关于时区，有以下几个标准：
*  CST：北美中部标准时间
*  UTC：协调世界时，又称世界标准时间，简称UTC，从英文国际时间/法文协调时间”Universal Time/Temps Cordonné”而来。中国大陆、香港、澳门、台湾、蒙古国、新加坡、马来西亚、菲律宾、澳洲西部的时间与UTC的时差均为+8，也就是UTC+8。
*  GMT：格林尼治标准时间（旧译格林威治平均时间或格林威治标准时间；英语：Greenwich Mean Time，GMT）是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。

Linux 的时间分为两种：
1. 硬件时间：由 BIOS（或CMOS）所负责。
2. 系统时间：由 Linux 所负责，系统时间在系统开关机后读取硬件时间后，再由 Linux 管理时间。

## date
date命令是显示或设置系统时间与日期。

这个是最简单、最直观获取系统时间与日期的方式了。
```
$ date
Thu Jul 30 13:23:50 CST 2020
```

显示所在时区：
```
date +"%Z %z"
CST +0800
```
> 注意 `+` 和 `"`之间没有空格，否则会报表。

date 命令常见参数：
```
%H 小时，24小时制（00~23）
%I 小时，12小时制（01~12）
%k 小时，24小时制（0~23）
%l 小时，12小时制（1~12）
%M 分钟（00~59）
%p 显示出AM或PM
%r 显示时间，12小时制（hh:mm:ss %p）
%s 从1970年1月1日00:00:00到目前经历的秒数
%S 显示秒（00~59）
%T 显示时间，24小时制（hh:mm:ss）
%X 显示时间的格式（%H:%M:%S）
%Z 以字符串的形式显示时区，日期域（CST）
%z 以数字的形式显示时区 (+0800)
%a 星期的简称（Sun~Sat）
%A 星期的全称（Sunday~Saturday）
%h,%b 月的简称（Jan~Dec）
%B 月的全称（January~December）
%c 日期和时间（Tue Nov 20 14:12:58 2012）
%d 一个月的第几天（01~31）
%x,%D 日期（mm/dd/yy）
%j 一年的第几天（001~366）
%m 月份（01~12）
%w 一个星期的第几天（0代表星期天）
%W 一年的第几个星期（00~53，星期一为第一天）
%y 年的最后两个数字（1999则是99）
```

## timedatectl
timedatectl 命令非常的方便，当你不带任何参数运行它时，这条命令可以像下图一样，输出系统时间概览，其中包含当前时区：

```
$ timedatectl

Local time: Thu 2020-07-30 05:30:21 UTC
                  Universal time: Thu 2020-07-30 05:30:21 UTC
                        RTC time: Thu 2020-07-30 05:30:21
                       Time zone: Etc/UTC (UTC, +0000)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

只查看时区：
```
$ timedatectl | grep "Time zone"
```

## /etc/timezone 
使用 cat 命令显示文件 `/etc/timezone` 的内容，来查看时区：

```
$ cat /etc/timezone
Etc/UTC
```

选择时区
```
$ tzselect
```

选择完成之后，将时区相关的配置，写入`.profit`配置文件中。

然后使用 souce 命令，强制生效。
```
souce .profit
```

### 参考链接
* [在 Linux 中查看时区](https://linux.cn/article-7970-1.html)
* [Linux date 命令](https://man.linuxde.net/date)
* [世界时钟地图](https://24timezones.com/map_zh.php#/map)