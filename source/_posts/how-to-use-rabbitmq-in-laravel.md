---
title: 如何在 Laravel 中使用 RabbitMQ
date: 2021-07-19 22:42:10
tags: ["PHP", "Laravel", "RabbitMQ"]
categories: ["PHP", "Laravel"]
---

这篇笔记用来记录如何在Laravel 中使用RabbitMQ。

<!-- more -->

Laravel 自带了那么多队列，为什么还要使用其他队列？

这是因为Laravel 的队列，通常是基于Redis、Database的Driver，使用起来有一定的局限性。

使用专业的队列，有如下优点：
1. 性能更好
2. 提供可靠性消息投递模式、返回模式
3. 保证数据不丢失的前提下做到高可靠性、可用性

如果还不知道Rabbit MQ 是什么，可以看一下我的另一篇笔记——[RabbitMQ 快速上手](https://www.0x2beace.com/rabbitmq-quick-start/)

## 安装
Laravel 并没有默认为我们提供 RabbitsMQ Driver，所幸一些勤劳的人，帮我们完成了一些艰苦的工作——[RabbitMQ Queue driver for Laravel](https://github.com/vyuldashev/laravel-queue-rabbitmq)。

引入依赖：
```bash
composer require vladimir-yuldashev/laravel-queue-rabbitmq
```

编辑`config/queue.php`，加入一个新的连接：
```php
'connections' => [
    // ...

    'rabbitmq' => [
    
       'driver' => 'rabbitmq',
       'queue' => env('RABBITMQ_QUEUE', 'default'),
       'connection' => PhpAmqpLib\Connection\AMQPLazyConnection::class,
   
       'hosts' => [
           [
               'host' => env('RABBITMQ_HOST', '127.0.0.1'),
               'port' => env('RABBITMQ_PORT', 5672),
               'user' => env('RABBITMQ_USER', 'guest'),
               'password' => env('RABBITMQ_PASSWORD', 'guest'),
               'vhost' => env('RABBITMQ_VHOST', '/'),
           ],
       ],
   
       'options' => [
           'ssl_options' => [
               'cafile' => env('RABBITMQ_SSL_CAFILE', null),
               'local_cert' => env('RABBITMQ_SSL_LOCALCERT', null),
               'local_key' => env('RABBITMQ_SSL_LOCALKEY', null),
               'verify_peer' => env('RABBITMQ_SSL_VERIFY_PEER', true),
               'passphrase' => env('RABBITMQ_SSL_PASSPHRASE', null),
           ],
           'queue' => [
               'job' => VladimirYuldashev\LaravelQueueRabbitMQ\Queue\Jobs\RabbitMQJob::class,
           ],
       ],
   
       /*
        * Set to "horizon" if you wish to use Laravel Horizon.
        */
       'worker' => env('RABBITMQ_WORKER', 'default'),
        
    ],

    // ...    
],
```

最后编辑`.env`，将`QUEUE_CONNECTION` 改成 `rabbitmq`，同时加入以下内容：
```php
RABBITMQ_HOST=127.0.0.1
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
```

至此，基本的配置工作就完成了，如果需要查看更多配置选项，可以查看[文档](https://github.com/vyuldashev/laravel-queue-rabbitmq#optional-config)。

RabbitMQ Queue 的使用是遵守Laravel 队列API的，也就是说，只需要将Driver 设置为 rabbitmq，我们根本无需关心底层的连接是如何实现的，就像正常使用Laravel 队列那样就好。

如果你不知道如何使用Laravel 队列API，请查阅[官方文档](http://laravel.com/docs/queues)。

## 基本使用

首先创建一个Job，来感受一下RabbitMQ：
```bash
php artisan make:job RabbitMQJob
```

入队：
```php
Route::get("rabbitmq", function () {
    dispatch(new \App\Jobs\RabbitMQJob())->onQueue("rabbitmq-job");
    
    return "ok";
});
```

请求一下 `127.0.0.1:8000/rabbitmq`，**生产/投递**一个**任务/消息**到`rabbitmq-job` 队列中。

然后打开RabbitMQ 管控台——`http://localhost:15672`，可以看到多出了一个名为`rabbitmq-job` 的队列：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718220527.png)

可以看到Ready 的数量是 1，表示该队列中等待消费的消息数量是 1，如果再次请求接口，就会发现Ready 的数量变成了 2。

此时如果没有消费者去主动消费，那么消息则一直存在于队列中。

尝试指定队列开始消费：
```php
php artisan queue:work --queue rabbitmq-job
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718220539.png)

这个就是消费者进行消费的过程。

再次查看管控台，可以发现Ready 的数量变成了 0。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210718220547.png)

这个就是RabbitMQ 在Laravel 中的基本使用，还是很容易上手的。

## 延时队列
队列的另一个常见的使用场景就是——延时队列。

在使用Redis 作为Driver 时，可以很轻松使用`delay` API 实现延迟任务。

但是对于RabbitMQ而言，如果只是调用`delay` API，就会发现消息不会被消费。

于是去查阅官方文档，看看是不是哪里的配置没有启用。但遗憾的发现，文档中并没有说明延时队列的使用方式。

最后抱着一丝侥幸在这个[Issues](https://github.com/vyuldashev/laravel-queue-rabbitmq/issues/342#issuecomment-659209409) 下找到了答案。

可以发现，决定因素就是下面这行代码：
```php
Artisan::call('rabbitmq:queue-declare', ['name' => $this->queue]);
```

这行代码的意思就是，调用一个Artisan 命令，而这个命令则会生成一个队列。

> 这个命令是哪里来的？

这是RabbitMQ Queue 扩展包封装的。

类似命令还有以下：
```php
rabbitmq:consume              Consume messages
rabbitmq:exchange-declare     Declare exchange
rabbitmq:exchange-delete      Delete exchange
rabbitmq:queue-bind           Bind queue to exchange
rabbitmq:queue-declare        Declare queue
rabbitmq:queue-delete         Delete queue
rabbitmq:queue-purge          Purge all messages in queue
```

## 结语
使用RabbitMQ 作为Laravel 队列驱动，使得Laravel 队列的可扩展性更高了。