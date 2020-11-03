---
title: Swoole 进程学习
date: 2020-11-01 20:53:56
tags: ["PHP", "Swoole"]
categories: ["PHP"]
---

记录Swoole 进程学习过程。

<!-- more -->

### 1. 创建一个进程
```
<?php
//  获取当前进程 ID
echo "我是 一个 主进程，我的ID是：" . posix_getpid().PHP_EOL;
// 为进程设置名称
cli_get_process_title("Master");

while (true) {
	sleep(1);
}
```

### 2. 创建一个子进程，如何回收子进程。
```
<?php
//  获取当前进程 ID
echo "我是 一个 主进程，我的ID是：" . posix_getpid().PHP_EOL;
// 为进程设置名称
cli_get_process_title("Master");

// 创建一个子进程
$child = new \Swoole\Process(function (){
	cli_get_process_title("Child");
	// 这是一个匿名函数，也就是定义子进程需要做的事情。
	echo "我是一个子进程，我的ID 是：" . posix_getpid() . PHP_EOL;
	// 如果就这样放着不管，那么这个子进程不会被回收，它是一个僵尸进程，虽然在那里但是并没有做事情，它的生命周期已经结束了。
});
// 创建
$child->start();

// 回收子进程
\Swoole\Process::wait();

while (true) {
	sleep(1);
}
```

### 3. 重定向子进程标准输出

子进程默认的标准输出是输出到屏幕上，可以通过对子进程设置，把输出重定向至管道。

然后再由主进程把管道中的内容读取出来。
```
<?php

//  获取当前进程 ID
echo "我是 一个 主进程，我的ID是：" . posix_getpid().PHP_EOL;
// 为进程设置名称
cli_get_process_title("Master");

// 创建一个子进程
$child = new \Swoole\Process(function (){
	cli_get_process_title("Child");

	while (true) {
		// 这是一个匿名函数，也就是定义子进程需要做的事情。
		echo "我是一个子进程，我的ID 是：" . posix_getpid() . PHP_EOL;
		// 如果就这样放着不管，那么这个子进程不会被回收，它是一个僵尸进程，虽然在那里但是并没有做事情，它的生命周期已经结束了。
		sleep(1);
	}
}, true);
// 创建
$child->start();

// 回收子进程，是否阻塞等待，默认为true，阻塞。
\Swoole\Process::wait(false);

while (true) {
	echo "通过主进程从管道中读取信息：". $child->read(). PHP_EOL;
	sleep(1);
}
```
这样做的好处是，可以通过主进程集中处理子进程的输出（比如可以写入日志），避免输出直接到屏幕中了。

第一个参数的作用是：是否将输出重定向至主进程。
true：将输出重定向至主进程管道。
false：直接将输出重定向至屏幕。

第二个参数的作用是：是否创建管道。
0：不创建
1. 创建Tcp 管道
2. 创建Udp 管道

第三个参数的作用是：是否启用协程。

### 4. 多个子进程的回收

如果主进程只是执行一次就退出，而子进程还一直在，那么主进程也不会直接退出。

如果有多个子进程，其中某一个子进程退出了，而另一个并没有退出，这时主进程也会选择退出，而剩余的那个子进程则成了僵尸进程。
因为它的父进程的ID 为零。

如果不做信号处理，否则子进程一旦退出，都会引起父进程退出。如果这时还有其他子进程没有退出，这会造成其他子进程变成僵尸进程。   

5. 在子进程中创建服务

分别是Master、Manager、Worker 进程，以及该子进程的父进程。

可以单独设置http 进程：
```
$http->set([
  "worker_num" => 1
]);
```

这样的话，进程就变成了两类：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201101204306.png)

最上面那个是父进程，下面三个分别是Master、Manger、Worker 进程。

### 6. 在进程中使用协程

### 7. 子进程使用管道进行通信
```
<?php
use \Swoole\Process;
//  引入协程
use \Swoole\Coroutine\Mysql as Mysql;

//  获取当前进程 ID
echo "我是 一个 主进程，我的ID是：" . posix_getpid().PHP_EOL;

$child = new Process(function (Process $proces){
	// $mysql = new \think\db\builder\Mysql();
	$mysql = new Mysql();
	$db = $mysql->connect(["host" => "127.0.0.1", "user" => "root", "password" => "122410", "database" => "2v"]);
	while (true) {
		$sql = "select * from 2v.2v_user where is_delete = 0 limit 0, 1";
		$rows = $mysql->query($sql);
		if ($rows) {
			$proces->write("我是一号子进程，正在查询数据：".$rows[0]["user_name"]);
		}
		sleep(1);
	}
}, false, 1, true);
// 创建子进程
$child->start();

$child2 = new Process(function (Process $process) {
	while (true) {
		sleep(1);
		$res = $process->read();
		if ($res) {
			echo "我是二号子进程，正在获取数据：".$res.PHP_EOL;
		}
	}
});
// 创建第二个子进程
$child2->start();

while (true) {
	// 一号子进程从管道中读取数据
	$data = $child->read();
	if ($data) {
		// 如果数据存在，二号子进程则向管道中写入数据
		$child2->write($data);
	}
	sleep(1);
}

// 通过信号回收子进程
Process::signal(SIGCHLD, function ($sig) {
	// 必须为false，非阻塞模式
	while ($res = Process::wait(false)) {
		echo "PID = {$res['pid']}";
	}
});
```

### 8. 子进程使用队列进行通信

### 9. 设置定时任务

通过Swoole 设置定时任务，到点之后自动执行定时任务。

核心逻辑：创建一个Manager 进程，通过一个while 循环，定时获取获取当前时间判断是否需要执行定时任务。

如果需要执行定时任务，则发送一个信号，在主进程中监听该信号， 然后执行对应的业务逻辑。

从 Swoole 4.x 版本开始，不再以监听信号的方式作为回收子进程了。