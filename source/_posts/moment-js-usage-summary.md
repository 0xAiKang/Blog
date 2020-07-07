---
title: moment.js 用法总结
date: 2020-07-07 18:59:32
tags: JavaScript
categories: 技术总结
---

## 前言
最近在做的一个前端项目，经常会遇到对时间的处理，因为原生的时间格式处理起来很费劲，所以引入了一个轻量级的日期处理类库。

[momentjs](http://momentjs.cn/) 支持日期格式化、Date、时间戳等相互转换，它使得操作时间变得非常简单。

### 快速上手
`momentjs`支持多个环境，所有的代码都应该在这两种环境中都可以工作。

#### Node.js

```
npm install moment
var moment = require('moment');
```

#### 浏览器
```
<script src="https://cdn.bootcss.com/moment.js/2.9.0/moment.js"></script>
```

### 实例
获取当前的日期和时间：
#### 创建
```
moment();
```
相当于moment(new Date()) 此处会返回一个moment封装的**日期对象**。

![](https://raw.githubusercontent.com/0xAiKang/CDN/master/blog/images/20200707190535.png)

#### 格式化
```
moment().format('YYYY年MM月DD日 HH:mm:ss') // "2020年07月07日 07:49:38"
moment().format('YYYY-MM-DD HH:mm:ss') // "2020-07-07 07:50:57"
moment().format('YYYY/MM/DD HH:mm:ss') // "2020/07/07 07:51:17"
moment().format('hh:m:ss') // "07:51:34"
moment().format('YYYY') // "2020"
moment().format('d') // 2，今天是周二
moment().format('X') // 获取当前时间的Unix时间戳
```

#### 转换为Date对象

```
moment().toDate() // Mon Jan 22 2018 18:11:55 GMT+0800 (中国标准时间)
moment('2018-01-20').toDate() // Tue Jan 20 2015 00:00:00 GMT+0800 (中国标准时间)
moment('2018-01-22 10:20:15').toDate() // Mon Jan 22 2018 10:20:15 GMT+0800 (中国标准时间)
moment(1448896064621).toDate() //毫秒转日期
```

#### 获取时间信息

```
moment().second() // 获取当前这一分钟的多少秒
moment().date() // 获取天
moment().day()  // 获取星期
moment().dayOfYear()  // 一年内的多少天
moment().week() // 一年里的多少周
moment().month()  // 获取当前月份（实际月份-1）
moment().quarter() // 一年内的第几个季度
moment().year() // 获取年份
moment().daysInMonth() // 获取当月天数
```

#### 显示
一旦解析和操作完成后，需要某些方式来显示 moment。

使用`format`来格式化日期：
```
moment().format() // "2020-07-07T08:24:35+08:00"
moment.unix(timestamp).format('YYYY-MM-DD HH:mm:ss');   // 将Unix 时间戳转换为日期格式
moment(timestamp).format('YYYY-MM-DD HH:mm:ss');   // 将Unix 毫秒时间戳转换为日期格式
moment().unix();        // 获取Unix 时间戳
moment().format("X");   // 获取Unix 时间戳
moment().format("x");   // 获取Unix 毫秒时间戳
```