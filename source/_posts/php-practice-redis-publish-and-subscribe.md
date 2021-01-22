---
title: PHP 实践 Redis 发布订阅
date: 2021-01-22 22:57:36
tags: ["PHP", "Redis"]
categories: ["PHP", "Redis"]
---

Redis 集成了Pub/Sub功能（means Publish, Subscribe）即发布及订阅功能。

<!-- more -->

Redis 有各种语言的客户端，这里仅以PHP 的客户端来了解Redis 的发布订阅。

发布者：即publish客户端，无需独占链接，你可以在publish消息的同时，使用同一个redis-client链接进行其他操作（例如：INCR等）

订阅者：即subscribe客户端，需要独占链接，即进行subscribe期间，redis-client无法穿插其他操作，此时client以阻塞的方式等待“publish端”的消息；
这一点很好理解，因此subscribe端需要使用单独的链接，甚至需要在额外的线程中使用。

## 终端实现

订阅者订阅频道：
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210122165931.png)

发布者向频道中发送内容
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210122170004.png)

## 代码实现

subscribe 客户端：
```
<?php
$redis = getConnect();
$redis->setOption(Redis::OPT_READ_TIMEOUT, -1); 

$redis->subscribe(["channel1"], function ($instance, $chan, $msg) {
	echo $msg;
	/**
	 * $instance 是上面创建的Redis 实例对象，因为独占链接的关系，该实例不能执行其他操作。
	 * 如果要使用Redis，需新建一个连接
	 */
	$redis = getConnect();
	$redis->get("name");
	// todo 业务逻辑
});

function getConnect()
{
	$redis = new Redis();
	$redis->connect("127.0.0.1", 6379);
	$redis->auth("");
	return $redis;
}
```

publish 客户端：
```
<?php
$redis = new Redis();
$redis->connect("127.0.0.1", 6379);

$redis->publish('channel1', 'hello, redis');
```

### 注意
subscribe 客户端需要手动设置不超时，有两种方式：
1. `ini_set('default_socket_timeout', -1)`
2. `$redis->setOption(Redis::OPT_READ_TIMEOUT, -1)`

如果不设置不超时，60s后会报一个错误：
```
Fatal error: Uncaught RedisException: read error on connection to 127.0.0.1:6379
```

方式一是通过临时修改 `php.ini` 配置项，`default_socket_timeout`默认为 60s 。

`default_socket_timeout` 是socket流的超时参数，即socket流从建立到传输再到关闭整个过程必须要在这个参数设置的时间以内完成，如果不能完成，那么PHP将自动结束这个socket并返回一个警告。

推荐第二种，因为方式二是通过修改redis的配置项，因此仅对redis连接生效，相对于方式一，不会产生意外的对其他部分的影响。

### 参考链接
* [php实现redis消息发布订阅](https://segmentfault.com/a/1190000020385114)