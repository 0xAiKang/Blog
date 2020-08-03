---
title: Supervisor 快速上手
date: 2020-07-30 20:38:38
tags: ["PHP", "进程管理", "Tutorial"]
categories: ["PHP", "Tutorial",]
---

[supervisord](http://supervisord.org/) 是一个用 Python 写的进程管理工具，是类Unix系统中的一个进程管理工具，

`Supervisor` 只适用于类Unix 系统，不适用于Window。

<!-- more -->

## 安装
因为`Supervisor` 是用 `Python` 所写的，所以可以直接使用`pip` 安装：

```
sudo pip install supervisor
```

Ubuntu：
```
apt-get install supervisor
```

Mac：
```
brew install supervisor
```

## 配置
`Supervisor`运行时会启动一个进程——`supervisord` 。
* `supervisord`：它负责启动所管理的进程，并将所管理的进程作为自己的子进程来启动，而且可以在所管理的进程出现崩溃时自动重启。
* `supervisorctl`：是命令行管理工具，可以用来执行 stop、start、restart 等命令，来对这些子进程进行管理。

查看默认配置项
```
$ echo_supervisord_conf
```

将默认配置项重定向至配置文件：
```
$ echo_supervisord_conf > /etc/supervisord.conf
```

然后可以看到 `/etc/` 配置文件下出现了以下文件，其中`/etc/supervisor` 是我们需要的配置文件。
```
$ find /etc/ -name supervisor
/etc/default/supervisor
/etc/init.d/supervisor
/etc/supervisor
```

`/etc/supervisord.conf` 核心配置文件，参考以下部分配置，`;` 表示注释。

因为`Supervisor`默认配置会把socket文件和pid守护进程生成在/tmp/目录下，/tmp/目录是缓存目录，所以我们需要手动换成`/var/run`目录。
```
[unix_http_server]
;file=/tmp/supervisor.sock   ; UNIX socket 文件，supervisorctl 会使用
file=/var/run/supervisor.sock   ; 修改为 /var/run 目录，避免被系统删除
;chmod=0700                 ; socket 文件的 mode，默认是 0700
;chown=nobody:nogroup       ; socket 文件的 owner，格式： uid:gid

;[inet_http_server]         ; HTTP 服务器，提供 web 管理界面
;port=127.0.0.1:9001        ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
;username=user              ; 登录管理后台的用户名
;password=123               ; 登录管理后台的密码

[supervisord]
;logfile=/tmp/supervisord.log ; 日志文件，默认是 $CWD/supervisord.log
logfile=/var/log/supervisor/supervisord.log ; 修改为 /var/log 目录，避免被系统删除
logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
logfile_backups=10           ; 日志文件保留备份数量默认 10
loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
;pidfile=/tmp/supervisord.pid ; pid 文件
pidfile=/var/run/supervisord.pid ; 修改为 /var/run 目录，避免被系统删除
nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ; 可以打开的进程数的最小值，默认 200

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
;serverurl=unix:///tmp/supervisor.sock ; 通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致
serverurl=unix:///var/run/supervisor.sock ; 修改为 /var/run 目录，避免被系统删除
;serverurl=http://127.0.0.1:9001 ; 通过 HTTP 的方式连接 supervisord

; 包含其他的配置文件
[include]
files = relative/directory/*.ini    ; 可以是 *.conf 或 *.ini
```



`/etc/supervisor/conf.d` 则是用来配置管理进程的配置文件，所有需要被`supervisor` 管理的进程都需要在这里先配置。
```
[program:demo]
command=php demo.php  // 需要执行队列的名称
directory= /var/www  // 命令执行的目录或者说执行 command 之前，先切换到工作目录 可以理解为在执行命令前会切换到这个目录 
process_name=%(process_num)02d // 默认为 %(program_name)s，即 [program:x] 中的 x这个是进程名，如果下面的numprocs参数为1的话，就不用管这个参数了，它默认值%(program_name)s也就是上面的那个program冒号后面的

numprocs=1          // 进程数量当不为1时的时候，就是进程池的概念，注意process_name的设置
autostart=true    // 是否自动启动
autorestart=true      // 程序意外退出是否自动重启
startsecs=1       // 自动重启间隔 
startretries=20   // 当进程启动失败后，最大尝试启动的次数。。当超过3次后，supervisor将把此进程的状态置为FAIL 默认值为3
redirect_stderr=true  // 如果为true，则stderr的日志会被写入stdout日志文件中  理解为重定向输出的日志
user=root   // 这个参数可以设置一个非root用户，当我们以root用户启动supervisord之后。我这里面设置的这个用户，也可以对supervisord进行管理 
stopsignal=INT
stderr_logfile=/var/log/supervisor/demo.err.log   // 子进程的stdout的日志路径 输出日志文件
stdout_logfile=/var/log/supervisor/demo.out.log   // 错误日志文件 当redirect_stderr=true。这个就不用
```

## 启动

```
$ supervisord -c /etc/supervisord.conf
```

### 常用命令整理

停止进程，program_name 为 [program:x] 里的 x
```
supervisorctl stop program_name
```

启动进程

```
supervisorctl start program_name
```

重启进程

```
supervisorctl restart program_name
```

结束所有属于名为 groupworker 这个分组的进程 (start，restart 同理)
```
supervisorctl stop groupworker:
```

结束 groupworker:name1 这个进程 (start，restart 同理)
```
supervisorctl stop groupworker:name1
```

停止全部进程，注：start、restart、stop 都不会载入最新的配置文件

```
supervisorctl stop all
```

载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程
```
supervisorctl reload
```

根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启
```
supervisorctl update
```

### 常见问题
#### unlinking stale socket /var/run/supervisor.sock

```
$ find / -name supervisor.sock
/run/supervisor.sock

$ unlink /run/supervisor.sock
```

### 参考链接
* ["unix:///tmp/supervisor.sock no such file" 错误处理](http://m.aluaa.com/articles/2019/01/02/1546398594207.html)
* https://segmentfault.com/a/1190000015768529
* [使用 supervisor 管理进程](http://liyangliang.me/posts/2015/06/using-supervisor/)
* [Python 进程管理工具 Supervisor 使用教程](https://www.cnblogs.com/restran/p/4854623.html)