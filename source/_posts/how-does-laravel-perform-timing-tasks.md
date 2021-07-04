---
title: Laravel 如何执行定时任务
date: 2021-07-04 22:37:03
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

Laravel 如何执行定义任务？或者说如何设计定义任务的执行？

<!-- more -->

第一个想到的肯定是设置 crontab 定时任务，但是 crontab 所做的事情通常是，每隔一个时间段执行一次某个命令。

假设我们现在的需求是，每天晚上的十二点，去清理一次数据库。

那么如何将这件事情与 crontab 的定时任务关联起来呢？难道要写一个PHP 脚本，交给 crontab 每天凌晨执行一次？

这当然是一个办法，但是在Laravel 中有更好的方案——[任务调度](https://learnku.com/docs/laravel/8.x/scheduling/9399)。

## 新建Artisan 命令

使用以下生成命令类：
```bash
$ php artisan make:command DataCleaning --command=system:data-cleaning
```

在`handle` 中，编写『数据清理』的逻辑。
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class CalculateActiveUser extends Command
{
    // 供我们调用命令
    protected $signature = 'larabbs:calculate-active-user';

    // 命令的描述
    protected $description = '生成活跃用户';

    // 最终执行的方法
    public function handle()
    {
        // 数据清理
    }
}
```

## 定时任务
Laravel 命令调度器允许我们在 Laravel 中对命令调度进行清晰流畅的定义，并且仅需要在服务器上增加一条 crontab 任务即可。

调度在 `app/Console/Kernel.php` 文件的 `schedule` 方法中定义。

设置系统定时任务：
```bash
crontab -e
// 填入以下内容
* * * * * php /your_project_absolute_path schedule:run >> /dev/null 2>&1
```
这里为了测试定时任务的执行，先设置每分钟进行观察。

`/dev/null 2>&1` 分为两部分进行理解：
1. `/dev/null`是系统黑洞，也就是 `>>` 之前执行的输出信息会全部丢进这个黑洞中
2. 在标准输出中，`stdin` 是 0，`stdout` 是 1，`stderr` 是 2，所以它将 `stderr` 全部导到 `stdout`，`stdout`又被导回 `/dev/null`，也就是不输出

所以，这两段加起来的结果就是 `crontab` 的执行不会有任何输出。

但调试阶段，还是建议先输出到一个已知文件，这样好确定定时任务是否有正常执行（如果任务执行异常，会看到异常信息）。

```bash
* * * * * php /your_project_absolute_path schedule:run >> /tmp/crontab.log
```

系统的定时任务已经设定好了，现在 crontab 将会每分钟调用一次 Laravel 命令调度器，当 `schedule:run` 命令执行时， Laravel 会评估你的计划任务并运行预定任务。

## 任务调度

最后我们注册Laravel 调度任务即可：
```php
// app/Console/Kernel.php

<?php
class Kernel extends ConsoleKernel
{
    protected function schedule(Schedule $schedule)
    {
        // 每分钟执行一次『数据清理』（进行调试）
        $schedule->command('system:data-cleaning')->everyMinute();
        
        // daily 每天执行一次『数据清理』
    }
}
```
调试时，仍使用一分钟，如果运行正常，之后将`everyMinute` 替换成 `daily` 即可。

做完以上操作之后，只需要等待一分钟，然后查看 `/tmp/crontab.log` 日志文件，是否有输出，如果能看到以下输出，则表示任务调度正常。

```
[2021-07-04T20:30:01+08:00] Running scheduled command: '/usr/local/Cellar/php@7.4/7.4.20/bin/php' 'artisan' system:data-cleaning > '/dev/null' 2>&1
```

Laravel 的任务调度，使得执行定时任务变得非常方便。