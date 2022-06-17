---
title: ThinkPHP6 自定义日志驱动
date: 2022-06-17 16:53:38
tags: ["PHP", "ThinkPHP"]
categories: ["PHP"]
---

最近接手一个项目，`ThinkPHP6.x` 写的，日志处理形同虚设，每次出了啥问题，第一时间也不知道问题出在哪，调试起来障碍很多。

<!-- more -->

ThinkPHP 6.0 在日志这一块，改动挺大了，直接砍掉了原来的请求信息部分。

![clipboard.png](inkdrop://file:OwdyJOIKO)

## 日志记录
ThinkPHP 对系统的日志按照级别来分类记录，按照 `PSR-3` 日志规范。除非是实时写入的日志，其它日志都是在当前请求结束的时候统一写入的 所以不要在日志写入之后使用exit等中断操作会导致日志写入失败。

日志的级别从低到高依次为：`debug`，`info`，`notice`，`warning`，`error`，`critical`，`alert`， `emergency`，ThinkPHP 额外增加了一个 `sql` 日志级别仅用于记录SQL日志（并且仅当开启数据库调试模式有效）。

## 设置日志记录级别

`config/log.php`
```php
<?php
use app\handle\Tp6Log;

return [
    // 日志记录级别
    'level'        => [],
];
```

* 当 level 为空时，记录所有级别
* 当 level 不为空时，只记录level中指定的错误级别

```php
public function index()
{
    // 当 level 配置为 notice、warning
    // 会记录
    Log::warning("这是第一段日志");  
    // 不会记录，因为 info 不在日志级别中
    Log::info("这是第二段日志");
}
```

## 单一日志
默认的 ThinkPHP 日志是写在当前日期(年月)目录下的，如(runtime/admin/log/202204/30.log)

设置单文件日志写入之后，所有日志则写入 `single.log` 文件中：
```php
<?php
use app\handle\Tp6Log;

return [
    // 单文件日志写入
    'single' => true
];
```

## 独立日志
```php
<?php
use app\handle\Tp6Log;

return [
    // 独立日志级别
    'apart_level'    => ['error', 'warning', "info"],
];
```

设置独立日志级别之后，不同类型的日志将会分别记录到对应类型的日志文件下：
```
// 设置独立日志级别之前
runtime/admin/log/202204/30.log

// 设置独立日志级别之后
runtime/admin/log/202204/30_warning.log
runtime/admin/log/202204/30_error.log
runtime/admin/log/202204/30_info.log
```

## 日志的写入时机
日志写入时机提供两种
* 实时写入
* 程序执行完后写入

```php
<?php
use app\handle\Tp6Log;

return [
    // 实时写入
    'realtime_write' => true,
];
```

## 日志通道
`ThinkPHP6.x` 日志类的一大特性就是日志级别支持指定通道写入，也就是可以实现自定义的日志记录，自定义日志驱动类，实现 `think\contract\LogHandlerInterface` 接口。

将 `config/log.php` 中通道 type 改成自定义驱动类即可。

`config.php`：
```php
<?php

// +----------------------------------------------------------------------
// | 日志设置
// +----------------------------------------------------------------------
use app\handle\Tp6Log;

return [
    // 默认日志记录通道
    'default'      => env('log.channel', 'file'),
    // 日志记录级别
    'level'        => [],
    // 日志类型记录的通道 ['error'=>'email',...]
    'type_channel' => [],
    // 关闭全局日志写入
    'close'        => false,
    // 全局日志处理 支持闭包
    'processor'    => null,
    // ThinkPHP对系统的日志按照级别来分类记录，按照PSR-3日志规范，日志的级别从低到高依次为：
    // debug, info, notice, warning, error, critical, alert, emergency
    // ThinkPHP额外增加了一个sql日志级别仅用于记录SQL日志（并且仅当开启数据库调试模式有效）。
    // 日志通道列表
    'channels'     => [
        'file' => [
            // 日志记录方式
            'type'           => 'File',
            // 'type'           => Tp6Log::class,
            // 日志保存目录
            // "path"           => "",
            // 如果没有设置路径，日志会输出到对应的应用目录下; 如果设置了路径，则所有的日志都会输出到该路径下
            'path'           => app()->getRuntimePath() . 'debug',
            // 单文件日志写入
            'single'         => false,
            // 独立日志级别
            'apart_level'    => ['error', 'sql',],
            // 最大日志文件数量
            'max_files'      => 0,
            // 使用JSON格式记录
            'json'           => false,
            // 文件大小
            'file_size'   	=> 	1024*1024*2,
            // 日志处理
            'processor'      => null,
            // 关闭通道日志写入
            'close'          => false,
            // 日志输出格式化
            'format'         => '[%s][%s] %s',
            // 是否实时写入
            'realtime_write' => true,
        ],
        // 其它日志通道配置
    ],
];

```

## 自定义日志驱动

我希望哪些信息能被记录？
* 请求记录
* SQL 执行记录
* 错误信息

`ThinkPHP 5.x 版本`，还存在请求记录的日志。类似下面的输出：
```log
[2022-06-16T11:38:42+08:00] 127.0.0.1 POST localhost/api/v1.index/index
[运行时间：0.508767s] [吞吐率：1.97req/s] [内存消耗：705.89kb] [文件加载：156]
[ HEADER ] array (
  'accept-encoding' => 'gzip, deflate',
  'referer' => 'https://servicewechat.com/wx8703b750b3e3c6dc/devtools/page-frame.html',
  'accept' => '*/*',
  'version' => '',
  'token' => 'ae42R9eUhyp3NUdW+AzKYOvEcr/UCwYLGA6nNMxrhT5AApsjJCpQuCX5WA2jePdKZBV1RP1Ws0/cK0KXxAjNDu/JRW4tqYPJ2vyNiiCwT6WbLe0Y2t7fo2P4sfRZrRSEnePSlJAetbU0afh5mzi9X6NztTd7fk4cBtGFxhFmTMmWynoL+HZ0DBGQ2VjwT82DjYbaUq+Ww4JUybQeSvpxcwqbEY3UR0L++gz+tQ',
  'user-agent' => 'Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1 wechatdevtools/1.05.2107090 MicroMessenger/8.0.5 Language/zh_CN webview/',
  'content-type' => 'application/json',
  'scene' => 'weixin',
  'content-length' => '46',
  'connection' => 'keep-alive',
  'host' => 'apiv2.smshw.local',
)
[ PARAM ] array (
)
```

ThinkPHP 6.0 则是直接砍掉了记录请求信息。

这里直接拿`ThinkPHP5.x` 的日志类源码进行修改，在 `ThinkPHP6.x` 中作为日志驱动：

```php
<?php
declare (strict_types=1);

namespace app\handle;

use DateTime;
use DateTimeZone;
use Exception;
use think\App;
use think\contract\LogHandlerInterface;
use think\facade\Request;

/**
 * 本地化调试输出到文件
 */
class Tp6Log implements LogHandlerInterface
{
    /**
     * 配置参数
     * @var array
     */
    protected $config = [
        'time_format' => 'c',
        'single' => false,
        'file_size' => 2097152,
        'path' => '',
        'apart_level' => [],
        'max_files' => 0,
        'json' => false,
        'json_options' => JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES,
        'format' => '[%s][%s] %s',
    ];

    protected $app;

    // 实例化并传入参数
    public function __construct(App $app, $config = [])
    {
        $this->app = $app;

        if (is_array($config)) {
            $this->config = array_merge($this->config, $config);
        }

        if (empty($this->config['format'])) {
            $this->config['format'] = '[%s][%s] %s';
        }

        if (empty($this->config['path'])) {
            $this->config['path'] = $app->getRuntimePath() . 'log';
        }

        if (substr($this->config['path'], -1) != DIRECTORY_SEPARATOR) {
            $this->config['path'] .= DIRECTORY_SEPARATOR;
        }
    }

    /**
     * 日志写入接口
     * @access public
     * @param array $log 日志信息
     * @return bool
     */
    public function save(array $log): bool
    {
        $destination = $this->getMasterLogFile();


        $path = dirname($destination);
        !is_dir($path) && mkdir($path, 0755, true);

        $info = [];

        // 日志信息封装
        $time = DateTime::createFromFormat('0.u00 U', microtime())->setTimezone(new DateTimeZone(date_default_timezone_get()))->format($this->config['time_format']);

        $request = Request::instance();
        //新增
        $requestInfo = [
            'ip' => $request->ip(),
            'method' => $request->method(),
            'host' => $request->host(),
            'uri' => $request->url()
        ];

        if(isset($log['sql'][0]) && strpos('CONNECT',$log['sql'][0])){

        }

        if (!$this->config['json']) {

            $debugInfo = [
                'param' => '[ PARAM ] ' . var_export($request->param(), true),
                'header' => '[ HEADER ] ' . var_export($request->header(), true)
            ];
            foreach ($debugInfo as $row) {
                array_unshift($info, $row);
            }

            // 增加额外的调试信息
            $runtime = round(microtime(true) - $this->app->getBeginTime(), 10);
            $reqs = $runtime > 0 ? number_format(1 / $runtime, 2) : '∞';
            $memory_use = number_format((memory_get_usage() - $this->app->getBeginMem()) / 1024, 2);
            $time_str = '[运行时间：' . number_format($runtime, 6) . 's] [吞吐率：' . $reqs . 'req/s]';
            $memory_str = ' [内存消耗：' . $memory_use . 'kb]';
            $file_load = ' [文件加载：' . count(get_included_files()) . ']';
            array_unshift($info, $time_str . $memory_str . $file_load);


            array_unshift($info, "---------------------------------------------------------------\r\n[{$time}] {$requestInfo['ip']} {$requestInfo['method']} {$requestInfo['host']}{$requestInfo['uri']}");
        }

        foreach ($log as $type => $val) {
            $message = [];
            foreach ($val as $msg) {
                if (!is_string($msg)) {
                    $msg = var_export($msg, true);
                }

                $message[] = $this->config['json'] ?
                    json_encode(['time' => $time, 'type' => $type, 'msg' => $msg], $this->config['json_options']) :
                    sprintf($this->config['format'], $time, $type, $msg);
            }

            if (true === $this->config['apart_level'] || in_array($type, $this->config['apart_level'])) {
                //这一句很关键，可以给mysql或者其他独立的日志，也加上请求和时间等信息
                array_unshift($message, "---------------------------------------------------------------\r\n[{$time}] {$requestInfo['ip']} {$requestInfo['method']} {$requestInfo['host']}{$requestInfo['uri']}");
                // 独立记录的日志级别
                $filename = $this->getApartLevelFile($path, $type);
                $this->write($message, $filename);
                continue;
            }

            $info[$type] = $message;
        }

        if ($info) {
            return $this->write($info, $destination);
        }

        return true;
    }

    /**
     * 获取主日志文件名
     * @access public
     * @return string
     */
    protected function getMasterLogFile(): string
    {

        if ($this->config['max_files']) {
            $files = glob($this->config['path'] . '*.log');

            try {
                if (count($files) > $this->config['max_files']) {
                    unlink($files[0]);
                }
            } catch (Exception $e) {
                //
            }
        }

        if ($this->config['single']) {
            $name = is_string($this->config['single']) ? $this->config['single'] : 'single';
            $destination = $this->config['path'] . $name . '.log';
        } else {

            if ($this->config['max_files']) {
                $filename = date('Ymd') . '.log';
            } else {
                $filename = date('Ym') . DIRECTORY_SEPARATOR . date('d') . '.log';
            }

            $destination = $this->config['path'] . $filename;
        }

        return $destination;
    }

    /**
     * 获取独立日志文件名
     * @access public
     * @param string $path 日志目录
     * @param string $type 日志类型
     * @return string
     */
    protected function getApartLevelFile(string $path, string $type): string
    {

        if ($this->config['single']) {
            $name = is_string($this->config['single']) ? $this->config['single'] : 'single';

            $name .= '_' . $type;
        } elseif ($this->config['max_files']) {
            $name = date('Ymd') . '_' . $type;
        } else {
            $name = date('d') . '_' . $type;
        }

        return $path . DIRECTORY_SEPARATOR . $name . '.log';
    }

    /**
     * 日志写入
     * @access protected
     * @param array $message 日志信息
     * @param string $destination 日志文件
     * @return bool
     */
    protected function write(array $message, string $destination): bool
    {
        // 检测日志文件大小，超过配置大小则备份日志文件重新生成
        $this->checkLogSize($destination);

        $info = [];

        foreach ($message as $type => $msg) {
            $info[$type] = is_array($msg) ? implode(PHP_EOL, $msg) : $msg;
        }


        $message = implode(PHP_EOL, $info) . PHP_EOL;

        return error_log($message, 3, $destination);
    }

    /**
     * 检查日志文件大小并自动生成备份文件
     * @access protected
     * @param string $destination 日志文件
     * @return void
     */
    protected function checkLogSize(string $destination): void
    {
        if (is_file($destination) && floor($this->config['file_size']) <= filesize($destination)) {
            try {
                rename($destination, dirname($destination) . DIRECTORY_SEPARATOR . time() . '-' . basename($destination));
            } catch (Exception $e) {
                //
            }
        }
    }
}

```

然后将 `config/log.php` 配置中的 type 设置为 `Tp6Log::class` ，便可以将请求记录、SQL 执行记录、错误等信息统统记录到日志文件中。

## 参考链接
* [thinkphp6自定义日志驱动,增加显示全部请求信息](https://blog.csdn.net/Function_JX_/article/details/121261012)