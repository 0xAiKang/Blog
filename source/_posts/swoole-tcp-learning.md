---
title: Swoole Tcp 学习
date: 2020-11-02 23:22:49
tags: ["PHP", "Swoole"]
categories: ["PHP"]
---

最近一直在学习Swoole，刚好有个老项目的一小部分有用到了Tcp 协议，借此机会重构一下。

<!-- more -->

### 场景描述：
该脚本的作用用一句话就可以概述：将本地数据源推送给另外一台服务器。

原始的处理方式，不合理的地方有以下几点：
1. 目标服务器需要开放指定端口，这会导致目标服务器向外暴露，不安全。
2. 如果有多台目标服务器，这会导致需要修改源码，脚本维护起来不方便。

### 重构
重构需要解决的问题有如下：
1. 当客户端连接成功后，才会向该客户端推送数据。
2. 当客户端断开连接时，停止向该客户端推送数据。
3. 允许多个客户端同时连接。
4. 因为数据源是不间断的，理论上只要客户端的连接不主动断开，服务端的数据推送就不会主动停止。

最终使用Swoole 的Tcp + Process 实现了以上需求，核心代码如下：
```
<?php
use Swoole\Process;

/**
 * 创建Server 对象，监听本地 9501 端口。
 */
$server = new Swoole\Server("0.0.0.0", 9501);

$server->set([
	"enable_coroutine" => false,
]);

$workers = [];

/**
 * 监听连接进入事件
 */
$server->on("Connect", function ($server, $fd) {
	global $workers;
	
	// 创建子进程
	$process = new swoole_process(function (swoole_process $worker) use ($server, $fd) {
		echo "Client Connect" . PHP_EOL;

	  // todo 业务逻辑
	  ...
	   
	  // 向客户端推送消息   
	  $server->send($fd, $str);
	  
	}, true);
	
	// 启动子进程
	$pid = $process->start();
	
	array_push($workers, ["pid" => $pid, "fd" => $fd]);
});

/**
 * 监听数据接收事件
 */
$server->on("Receive", function ($server, $fd, $from_id, $data){
	$server->send($fd, "Server: " . $data);
});

/**
 * 监听连接关闭事件
 */
$server->on("Close", function ($server, $fd) {
	global $workers;
	
	foreach ($workers as $worker) {
	  if ($item['fd'] === $fd){
	    // 检查子进程是否存在
  		if (Process::kill($worker['pid'], 0)){
  			array_shift($workers);
  			// 通过信号终止子进程
  			Process::kill($item['pid'], SIGKILL);
  		}
	  }
	}
	echo "Client Close" . PHP_EOL;
});

// 启动TCP 服务器
$server->start();
```

其实实现的原理很简单，利用Swoole 的基于事件的 Tcp 异步编程，当有客户端连接时，就创建一个子进程进行推送数据，但客户端连接断开时，就通过信号结束该客户端对应的子进程。