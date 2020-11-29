---
title: Swoole 协程学习
date: 2020-11-29 22:36:47
tags: ["PHP", "Swoole"]
categories: ["PHP"]
---

第一次接触协程这个概念，是在学习Swoole时，那时看官方文档并不能完全理解协程到底是个什么东西以及该如何正确的使用它。

<!-- more -->

后来逐渐看了一些写的比较通俗的文章，加上自己的一些理解，逐步开始对协程有一些认识了。

## 认识协程
**协程不是进程或线程**，其执行过程更类似于子例程，或者说不带返回值的函数调用。

上面那句话很关键，一句话就把协程是什么，不是什么说清楚了。

下面这张图可以很清晰的看到协程与多进程的区别：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201129093350.png)

### 执行顺序
下面这段代码主要做了三件事：写入文件、发送邮件以及插入数据。

```
<?php
function task1(){
    for ($i=0;$i<=300;$i++){
        //写入文件,大概要3000微秒
        usleep(3000);
        echo "写入文件{$i}\n";
    }
}
function task2(){
    for ($i=0;$i<=500;$i++){
        //发送邮件给500名会员,大概3000微秒
        usleep(3000);
        echo "发送邮件{$i}\n";
    }
}
function task3(){
    for ($i=0;$i<=100;$i++){
        //模拟插入100条数据,大概3000微秒
        usleep(3000);
        echo "插入数据{$i}\n";
    }
}
task1();
task2();
task3();
```

这段代码和上面不同的是，这三件事情是交叉执行的，每个任务执行完一次之后，切换到另一个任务，如此循环。

类似于这样的执行顺序，就是协程。
```
<?php
function task1($i)
{
	//使用$i标识 写入文件,大概要3000微秒
	if ($i > 300) {
		return false;//超过300不用写了
	}
	echo "写入文件{$i}\n";
	usleep(3000);
	return true;
}

function task2($i)
{
	//使用$i标识 发送邮件,大概要3000微秒
	if ($i > 500) {
		return false;//超过500不用发送了
	}
	echo "发送邮件{$i}\n";
	usleep(3000);
	return true;
}

function task3($i)
{
	//使用$i标识 插入数据,大概要3000微秒
	if ($i > 100) {
		return false;//超过100不用插入
	}
	echo "插入数据{$i}\n";
	usleep(3000);
	return true;
}

$i = 0;
while (true) {
	$task1Result = task1($i);
	$task2Result = task2($i);
	$task3Result = task3($i);
	if($task1Result===false&&$task2Result===false&&$task3Result===false){
		break;//全部任务完成,退出循环
	}
	$i++;
}
```

swoole实现协程代码：
```
<?php
function task1(){
    for ($i=0;$i<=300;$i++){
        //写入文件,大概要3000微秒
        usleep(3000);
        echo "写入文件{$i}\n";
        Co::sleep(0.001);//挂起当前协程,0.001秒后恢复//相当于切换协程
    }
}
function task2(){
    for ($i=0;$i<=500;$i++){
        //发送邮件给500名会员,大概3000微秒
        usleep(3000);
        echo "发送邮件{$i}\n";
        Co::sleep(0.001);//挂起当前协程,0.001秒后恢复//相当于切换协程
    }
}
function task3(){
    for ($i=0;$i<=100;$i++){
        //模拟插入100条数据,大概3000微秒
        usleep(3000);
        echo "插入数据{$i}\n";
        Co::sleep(0.001);//挂起当前协程,0.001秒后恢复//相当于切换协程
    }
}
$pid1 = go('task1');//go函数是swoole的开启协程函数，用于开启一个协程
$pid2 = go('task2');
$pid3 = go('task3');
```

### 协程与多进程
由上面的代码，可以发现，协程其实只是运行在一个进程中的函数，只是这个函数会被切换到下一个执行。

> 需要注意的是⚠️：

协程并不是多任务并行处理，它属于多任务串行处理，它俩的本质区别是在某个时刻同时执行一个还是多个任务。

### 协程的作用域
由于协程就是进程中一串任务代码，所以它的全局变量、静态变量等变量都是共享的，包括 PHP 的全局缓冲区。

所以在开发时特别需要注意作用域相关的问题。

### 协程的I/O连接
在协程中，要特别注意不能共用一个 I/O 连接，否则会造成数据异常。

由于协程的交叉运行机制，且各个协程的 I/O 连接都必须是相互独立的，这时如果使用传统的直接建立连接方式，会导致每个协程都需要建立连接、闭关连接，从而消耗大量资源。那么该如何解决协程的 I/O 连接问题呢？这个时候就需要用到连接池了。

连接池存在的意义在于，复用原来的连接，从而节省重复建立连接所带来的开销。

### 协程的实际应用场景
说了这么多，那协程倒底能解决哪些实际业务场景呢？下面通过一个实例来快速上手协程（笔者当时写这篇文章时，对协程的理解还不够深刻，所以这里引用[zxr615](https://learnku.com/blog/zxr615) 的”[做饭](https://learnku.com/articles/44836)“的例子来理解协程）：

传统同步阻塞实现逻辑：
```
<?php
function cook()
{
	$startTime = time();

	echo "开始煲汤..." . PHP_EOL;
	sleep(10);
	echo "汤好了..." . PHP_EOL;

	echo "开始煮饭..." . PHP_EOL;
	sleep(8);
	echo "饭熟了..." . PHP_EOL;

	echo "放油..." . PHP_EOL;
	sleep(1);
	echo "煎鱼..." . PHP_EOL;
	sleep(3);
	echo "放盐..." . PHP_EOL;
	sleep(1);
	echo "出锅..." . PHP_EOL;

	var_dump('总耗时：' . (time() - $startTime) . ' 分钟');
}

cook();
```

协程实现逻辑：
```
<?php
use Swoole\Coroutine;
use Swoole\Coroutine\WaitGroup;
use Swoole;

class Cook
{
	public function cookByCo()
	{
		$startTime = time();

		// 开启一键协程化: https://wiki.swoole.com/#/runtime?id=swoole_hook_all
		Swoole\Runtime::enableCoroutine($flags = SWOOLE_HOOK_ALL);

		// 创建一个协程容器: https://wiki.swoole.com/#/coroutine/scheduler
		// 相当于进入厨房
		\Co\run(function () {
			// 等待结果: https://wiki.swoole.com/#/coroutine/wait_group?id=waitgroup
			// 记录哪道菜做好了，哪道菜还需要多长时间
			$wg = new WaitGroup();
			// 保存数据的结果
			// 装好的菜
			$result = [];

			// 记录一下煲汤(记录一个任务)
			$wg->add();
			// 创建一个煲汤任务(开启一个新的协程)
			Coroutine::create(function () use ($wg, &$result) {
				echo "开始煲汤..." . PHP_EOL;
				// 煲汤需要6分钟，所以我们也不用在这里等汤煮好，
				// 直接去做下一个任务：炒菜(协程切换)
				sleep(8);
				echo "汤好了..." . PHP_EOL;

				// 装盘
				$result['soup'] = '一锅汤';
				$wg->done(); // 标记任务完成
			});

			// 记录一下煮饭(记录一个任务)
			$wg->add();
			// 创建一个煮饭任务(开启一个新的协程)
			Coroutine::create(function () use ($wg, &$result) {
				echo "开始煮饭..." . PHP_EOL;
				// 煮饭需要5分钟，所以我们不用在这里等饭煮熟，放在这里一会再来看看好了没有
				// 我们先去煲汤(协程切换)
				sleep(10);
				echo "饭熟了..." . PHP_EOL;

				// 装盘
				$result['rice'] = '一锅米饭';
				$wg->done(); // 标记任务完成
			});

			// 记录一下炒菜
			$wg->add();
			// 创建一个炒菜任务(再开启一个新的协程)
			Coroutine::create(function () use ($wg, &$result) {
				// 煎鱼的过程必须放在一个协程里面执行，如果不是的话可能鱼还没煎好就出锅了
				// 因为开启协程后，IO全是异步了，在此demo中每次遇到sleep都会挂起当前协程
				// 切换到下一个协程执行。
				// 例如把出锅这一步开启一个新协程执行，则在煎鱼的时候鱼，鱼就出锅了。
				echo "放油..." . PHP_EOL;
				sleep(1);
				echo "煎鱼..." . PHP_EOL;
				sleep(3);
				echo "放盐..." . PHP_EOL;
				sleep(1);
				echo "出锅..." . PHP_EOL;

				// 装盘
				$result['food'] = '鱼香肉丝';
				$wg->done();
			});

			// 等待全部任务完成
			$wg->wait();

			// 返回数据(上菜！)
			var_dump($result);
		});

		var_dump('总耗时：' . (time() - $startTime) . ' 分钟');
	}
}
$cooker = new Cook();
$cooker->cookByCo();
```

通过执行代码可以看到协程方式比传统阻塞方式足足快了十三分钟。从协程方式实现的逻辑中可以看到，通过无感知编写”同步代码“，却实现了异步 I/O 的效果和性能。避免了传统异步回调所带来的离散的代码逻辑和陷入多层回调中导致代码无法维护。

不过需要注意的是传统回调的触发条件是**回调函数**，而协程切换的条件是**遇到 I/O**。

### 协程误区
实际使用协程时，需要注意以下几个误区，否则效果可能会事倍功半。

理论上来讲，协程解决的是 I/O 复用的问题，对于计算密集的问题无效。
* 如果cpu很闲(大部分时间都消耗在网络磁盘上了)，协程就可以提高cpu的利用率
* 如果cpu本身就很饱和了 用协程反而会降低cpu利用率（需要花时间来做协程调度）。
* swoole 是单线程

### 参考链接
* [swoole 学习笔记-做一顿饭来理解协程](https://learnku.com/articles/44836)
* [协程-EasySwoole](https://www.easyswoole.com/Cn/NoobCourse/coroutine.html)
* [swoole 协程-swoole 高手之路](https://xiaoxiami.gitbook.io/swoole/swoole-ji-chu/jin-cheng-nei-cun-xie-cheng/swoole-xie-cheng)
* [swoole一个协程问题？为什么效率变慢了](https://segmentfault.com/q/1010000021755294)