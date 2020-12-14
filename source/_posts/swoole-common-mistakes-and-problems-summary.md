---
title: Swoole 常见误区及问题总结
date: 2020-12-14 22:12:33
tags: ["PHP", "Swoole"]
categories: ["PHP"]
---

随着对Swoole 的逐步了解，总结以下可能会碰到的误区：

<!-- more -->

1. Swoole 是单线程
2. Swoole 异步回调模块仅可用于 CLI 命令行模式
3. Swoole 只有同步阻塞的客户端才可在 `php-fpm` 中使用
4. Swoole 重新编译安装会自动覆盖掉之前的版本 
5. CPU密集型任务（科学计算等）, 不会引起协程的调度; IO密集型任务（网络请求, 文件读写等）, 才会引发协程的调度
6. `enable_coroutine` 开启协程支持之后，无需使用 `Co\Run` 创建协程
7. 所有的[协程](https://wiki.swoole.com/#/coroutine)必须在[协程容器](https://wiki.swoole.com/#/coroutine/scheduler)里面[创建](https://wiki.swoole.com/#/coroutine/coroutine?id=create)，Swoole 程序启动的时候大部分情况会自动创建协程容器
8. `Swoole\Coroutine` 前缀的类名映射为 Co。使用 `Co\Run` 方法创建协程容器，使用 `Coroutine::create` 或 `go` 方法创建协程。

常见问题：
* [什么是协程容器？](https://wiki.swoole.com/#/coroutine?id=%E4%BB%80%E4%B9%88%E6%98%AF%E5%8D%8F%E7%A8%8B%E5%AE%B9%E5%99%A8)
* [是否可以共用 1 个 Redis 或 MySQL 连接](https://wiki.swoole.com/#/question/use?id=%E6%98%AF%E5%90%A6%E5%8F%AF%E4%BB%A5%E5%85%B1%E7%94%A81%E4%B8%AAredis%E6%88%96mysql%E8%BF%9E%E6%8E%A5)
* [Call to undefined function Co\Run()](https://wiki.swoole.com/#/question/use?id=call-to-undefined-function-corun)