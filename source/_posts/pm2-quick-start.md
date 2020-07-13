---
title: PM2 快速上手
date: 2020-07-12 21:44:47
tags: ["PM2","Node", "Tutorial"]
categories: ["Node", "Tutorial"]
---

[PM2](http://pm2.keymetrics.io/) 是Node.js 生产环境中的进程管理工具，自带负载均衡功能。

<!-- more -->
安装
```
$ npm install pm2 -g
```
无缝更新
```
$ pm2 update
```
## 启动应用
PM2 中有两种方式启动应用，一种是**直接调用应用入口文件**，一种是**通过调用配置文件启动应用**。

### 命令行启动
在生产环境中，通过命令行启动服务
```
$ pm2 stat app.js
```

### 配置文件启动
很多时候，仅仅只是使用 `PM2` 去启动应用，可能不能完全满足我们的需求。

当需要对应用有更多的要求时，这个时候就需要用到`PM2` 的配置文件了。

PM2 支持通过配置文件创建管理应用，首先在项目根目录创建配置文件`precesses.json`：
```
{
  "apps": [
    {
      "name": "myApp",
      "cwd": "/var/www/app/",
      "script": "./app.js",
      "watch": true
    }
  ]
}
```
上面是一个最简单的`processes.json`配置，创建了一个`myApp`应用，如果你有多个服务，那么`apps` 这个数组中创建多个应用。

> 创建好配置文件之后，那么该如何启动呢？

有两种方式：
1. 直接调用配置文件启动

```
$ pm2 start processes.json
```
可以增加`--env`参数，来指定当前启动环境。

2. 通过`package.json` 配置文件，配置脚本启动

```
// package.json

"scripts": {
    "start": "node server/index",
    "pm2": "pm2 start processes.json"
  }
```

然后就可以直接使用`npm start pm2` 来启动应用了。

#### 参数说明
在配置文件你可以指定环境变量、日志文件、进程文件，重启最大次数…等配置项。支持JSON和YAML格式。

PM2 的配置支持非常多的参数，下面会对常用的参数一一做说明。

|字段|类型|值|描述|
|-|-|-|-|
|name|string|myApp|应用的名字，默认是脚本文件名|
|cwd|string|/var/www/myApp|应用程序所在目录|
|script|string|./server.js|应用程序的脚本路径，相对于应用程序所在目录|
|log_date_format|string|YYYY-MM-DD HH:mm Z|日志时间格式|
|error_file|string|-|错误日志存放路径|
|out_file|string|-|输出日志存放路径|
|pid_file|string|-|pid文件路径|
|watch|boolean or array|true|当目录文件或子目录文件有变化时自动重新加载应用|
|ignore_watch|list|[”[\/\]./”, “node_modules”]|list中的正则匹配的文件和目录有变化时不重新加载应用|
|max_memory_restart|string|50M|当应用超过设定的内存大小就自动重启|
|min_uptime|string|60s|最小运行时间，这里设置的是60s即如果应用程序在60s内退出，pm2会认为程序异常退出，此时触发重启max_restarts设置数量|
|max_restarts|number|10|设置应用程序异常退出重启的次数，默认15次（从0开始计数）|
|instances|number|1|启动实例个数|
|cron_restart|string|1 0 * * *|定时重启|
|exec_interpreter|string|node|应用程序的脚本类型，默认是node|
|exec_mode|string|fork|应用启动模式，支持fork和cluster模式，默认为fork|
|autorestart|boolean|true|应用程序崩溃或退出时自动重启|

> 注意：如果`processes.json`配置文件如果发生了变化，那么需要删除应用之后，重新创建，配置文件才会生效。

### 进程监控

列出所有节点应用程序（进程/微服务）

```
$ pm2 list
$ pm2 ls
```

可以将进程列表以JSON格式打印出来：
```
$ pm2 jlist
$ pm2 prettylist
```

使用进程ID或名称查看所示的单个Node进程的详细信息：
```
$ pm2 describe <id | app_name>
$ pm2 show <id | app_name>
```

实时监控所有进程CPU或内存使用情况：
```
$ pm2 monit
```

### 日志管理
查看某个应用的日志：

```
$ pm2 logs ['all' | app_name | app_id ]
```

```
$ pm2 logs --json         # JSON 格式输出
$ pm2 logs --format       # 格式化 output
$ pm2 flush               # 清空所有日志文件
$ pm2 reloadLogs          # 重新加载所有日志文件
```

### 常用命令

停止所有进程
```
$ pm2 stop all
```

重启所有进程

```
$ pm2 restart all
```

0秒停机重载进程 (用于 NETWORKED 进程)
```
$ pm2 reload all
```

停止指定的进程
```
$ pm2 stop 0
```

重启指定的进程
```
$ pm2 restart 0
```

杀死指定的进程
```
$ pm2 delete 0
```
杀死全部进程

```
$ pm2 delete all
```

### 使用PM2 运行 npm start
 
`npm run xxxx` 是 node常用的启动方式之一，那么如何使用`PM2`来实现对该方式的启动呢？

`npm run`、`npm start`等命令之所以可以使用，是因为`package.json`配置文件中增加了对应的脚本命令。

```
"scripts": {
    "start-dev": "env $(cat .env | xargs) nodemon server/index",
    "start": "node server/index",
  }
```

语法：
```
pm2 start npm --watch --name <taskname> -- run <scriptname>;
```
其中 `--watch`监听代码变化，`--name`重命名任务名称，`-- run`后面跟脚本名字

实例：
```
// 等效于 npm start
pm2 start npm --watch --name webserver -- run start
```

### 参考链接
* [如何在生产服务器上安装PM2运行Node.js应用程序](https://www.linuxidc.com/Linux/2019-07/159432.htm)
* [PM2 实用手册](https://fynn90.github.io/2018/01/11/PM2%E5%AE%9E%E7%94%A8%E6%89%8B%E5%86%8C/#%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4)
* [PM2 部署 nodejs 项目](https://www.cnblogs.com/hai-cheng/articles/8690115.html)