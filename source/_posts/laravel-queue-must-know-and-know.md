---
title: Laravel Queue 必知必会
date: 2021-07-17 22:36:23
tags: ["PHP", "Laravel", "MQ"]
categories: ["PHP", "Laravel", "MQ"]
---

这篇笔记转载于——[使用 Laravel Queue 不得不明白的知识](http://lijinma.com/blog/2017/01/31/laravel-queue/)。

<!-- more -->

我觉得这篇文章就属于就那种写得比较好的文章，原因如下：

授人鱼不如授人以渔。一般的文章通常只是教你如何使用，用完之后，你定是知其然不知其所以然。而这篇文章则是从“底层原理”的角度，去了解Laravel Queue。

## 命令详解

使用过Laravel Queue 的你，一定也会对这个命令感到很熟悉，通常我们需要对某个队列进行消费时，会执行这个命令。
```bash
php artisan queue:work --daemon --quiet --queue=default --delay=3 --sleep=3 --tries=3
```

这个命令有很多参数，一起来看看吧：
* `--daemon`：以守护进程的方式运行队列，通常会在生产环境中使用到。
* `--quiet`：不输出任何内容
* `--delay=3`：一个任务失败后，延迟多长时间后再重试（单位是秒）
* `--sleep=3`：去队列中消费时，如果发现没有任务，休息多长时间（单位是秒）
* `--tries=3`：定义失败任务最多重试次数

## 当我们 `dispatch` 一个`Job` 之后，倒底发生了哪些事情

这里为了方便调试及理解，需要先将 Queue driver 设置为`redis`。

```php
QUEUE_CONNECTION=redis
```

首先得创建一个Job：
```bash
php artisan make:job ExampleJob 
```

进入`redis-cli`，执行如下命令：
```bash
127.0.0.1:6379> monitor
OK
```

然后打开Tinker，分配一个任务：
```php
dispatch(new \App\Jobs\ExampleJob());
```

再次观察redis-cli 控制台。
```bash
1626490482.711473 [0 127.0.0.1:64056] "SELECT" "0"
1626490482.719953 [0 127.0.0.1:64056] "EVAL" "-- Push the job onto the queue...\nredis.call('rpush', KEYS[1], ARGV[1])\n-- Push a notification onto the \"notify\" queue...\nredis.call('rpush', KEYS[2], 1)" "2" "queues:default" "queues:default:notify" "{\"uuid\":\"d945da70-8c68-47d7-86c6-f631a4da6296\",\"displayName\":\"App\\\\Jobs\\\\ExampleJob\",\"job\":\"Illuminate\\\\Queue\\\\CallQueuedHandler@call\",\"maxTries\":null,\"maxExceptions\":null,\"failOnTimeout\":false,\"backoff\":null,\"timeout\":null,\"retryUntil\":null,\"data\":{\"commandName\":\"App\\\\Jobs\\\\ExampleJob\",\"command\":\"O:19:\\\"App\\\\Jobs\\\\ExampleJob\\\":10:{s:3:\\\"job\\\";N;s:10:\\\"connection\\\";N;s:5:\\\"queue\\\";N;s:15:\\\"chainConnection\\\";N;s:10:\\\"chainQueue\\\";N;s:19:\\\"chainCatchCallbacks\\\";N;s:5:\\\"delay\\\";N;s:11:\\\"afterCommit\\\";N;s:10:\\\"middleware\\\";a:0:{}s:7:\\\"chained\\\";a:0:{}}\"},\"id\":\"OjbqA9m9x7VsdzNds6JXImwYI86j0E9H\",\"attempts\":0}"
1626490482.725191 [0 lua] "rpush" "queues:default" "{\"uuid\":\"d945da70-8c68-47d7-86c6-f631a4da6296\",\"displayName\":\"App\\\\Jobs\\\\ExampleJob\",\"job\":\"Illuminate\\\\Queue\\\\CallQueuedHandler@call\",\"maxTries\":null,\"maxExceptions\":null,\"failOnTimeout\":false,\"backoff\":null,\"timeout\":null,\"retryUntil\":null,\"data\":{\"commandName\":\"App\\\\Jobs\\\\ExampleJob\",\"command\":\"O:19:\\\"App\\\\Jobs\\\\ExampleJob\\\":10:{s:3:\\\"job\\\";N;s:10:\\\"connection\\\";N;s:5:\\\"queue\\\";N;s:15:\\\"chainConnection\\\";N;s:10:\\\"chainQueue\\\";N;s:19:\\\"chainCatchCallbacks\\\";N;s:5:\\\"delay\\\";N;s:11:\\\"afterCommit\\\";N;s:10:\\\"middleware\\\";a:0:{}s:7:\\\"chained\\\";a:0:{}}\"},\"id\":\"OjbqA9m9x7VsdzNds6JXImwYI86j0E9H\",\"attempts\":0}"
1626490482.726123 [0 lua] "rpush" "queues:default:notify" "1"
```

正常可以看到以上输出，说明我们的Job 已经成功放入队列中了。

```bash
127.0.0.1:6379> keys queue*
1) "queues:default"
2) "queues:default:notify"
```

此时，如果执行`php artisan work`，则开始会消费队列：
```bash
php artisan queue:work --queue=default                                               
[2021-07-17 11:36:46][hpDVkINqhtLCi9jKqQKveC6utwX3C8jS] Processing: App\Jobs\ExampleJob
[2021-07-17 11:36:46][hpDVkINqhtLCi9jKqQKveC6utwX3C8jS] Processed:  App\Jobs\ExampleJob
```

通过分析上面的输出，可以知道队列在Redis 的消费过程应该是：
```php
//  首先从 queue:default List 中取出任务
"lpop" "queues:default"

// 暂存到queues:default:reserved Zset 中
"zadd" "queues:default:reserved" "1626493456" 

// 任务执行完毕后， 从 queues:default:reserved Zset 中删除
"zrem" "queues:default:reserved"

// 如果任务失败，会放到 queue:default:delay zset 中
```

转载文章是作者在Laravel `5.x` 版本时写的，时至如今，Laravel 已发布至`8.x` ，队列消费的细节可能发生了一些变化，但是核心的逻辑还是没变。