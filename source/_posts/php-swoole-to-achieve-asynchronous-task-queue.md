---
title: PHP + Swoole 实现异步任务队列
date: 2020-11-26 20:33:48
tags: ["PHP", "Swoole"]
categories: ["PHP"]
---
最近接手一个对接短信的需求，这个需求本身并没有什么难度，直接按照服务商的要求请求具体的接口就好了。

<!-- more -->

最开始是使用传统的同步阻塞方式实现了一遍，用户体验并不好，发送短信需要等待，等待服务商的接口返回内容，才继续向下执行。

因为最近在学习[Swoole](https://swoole.com/)，Swoole 中有一个“[异步任务](https://wiki.swoole.com/#/start/start_task?id=%E6%89%A7%E8%A1%8C%E5%BC%82%E6%AD%A5%E4%BB%BB%E5%8A%A1task)”，就特别适合以下应用场景：
1. 需要执行耗时操作，会阻塞主进程
2. 用户不需要等待返回结果

结合官网手册和[Latent](https://learnku.com/blog/pltrue) 的[基于 swoole 下 异步消息队列 API](https://learnku.com/articles/43752)，最终简单封装了一个处理API 的类：

### 服务端

服务端是基于本地Tcp，监听`9501`端口。
```
<?php
class taskServer{
	const HOST = "127.0.0.1";
	const PORT = 9501;
	public $server = null;

	public function __construct()
	{
		$this->server = new SWoole\Server(self::HOST, self::PORT);
		$this->server->set(array(
			"enable_coroutine" => false,     // 关闭协程
			"worker_num" => 2,               // 开启的进程数 一般为cup核数 1-4 倍
			"task_worker_num" => 2,          // task进程的数量
			'daemonize' => true,             // 以守护进程的方式启动
		));

		// 注册事件
		$this->server->on("connect", [$this, "onConnect"]);
		$this->server->on("receive", [$this, "onReceive"]);
		$this->server->on("close", [$this, "onClose"]);
		$this->server->on("task", [$this, "onTask"]);
		$this->server->on("finish", [$this, "onFinish"]);

		// 启用服务
		$this->server->start();
	}

	/**
	 * 监听连接事件
	 * @param $server
	 * @param $fd
	 */
	public function onConnect($server, $fd){
		echo "连接成功".PHP_EOL;
	}

	/**
	 * 监听客户端发送的消息
	 * @param $server       "Server 对象"
	 * @param $fd           "唯一标示"
	 * @param $form_id
	 * @param $data         "客户端发送的数据"
	 */
	public function onReceive($server, $fd, $form_id, $data){
		// 投递任务
		$server->task($data);
		$server->send($fd, "这是客户端向服务端发送的信息：{$data}");
	}

	/**
	 * 监听异步任务task事件
	 * @param $server
	 * @param $task_id
	 * @param $worker_id
	 * @param $data
	 * @return string
	 */
	public function onTask($server, $task_id, $worker_id, $data){
		$data = json_decode($data, true);
		echo "开始执行异步任务".PHP_EOL;
		try {
			// 开始执行任务
			$this->addLog(date('Y-m-d H:i:s')."开始执行任务".PHP_EOL );
			// 通知worker（必须 return，否则不会调用 onFinish）
			return $this->curl($data['url'], $data['data'], $data['type']);
		} catch (Exception $exception) {
			// 执行任务失败
			$this->addLog(date('Y-m-d H:i:s')."执行任务失败".PHP_EOL);
		}
	}

	/**
	 * 监听finish 事件
	 * @param $server
	 * @param $task_id
	 * @param $data
	 */
	public function onFinish($server, $task_id, $data){
		$this->addLog(date("Y-m-d H:i:s")."异步任务执行完成".PHP_EOL);
		print_r( "来自服务端的消息：{$data}");
	}

	/**
	 * 监听关闭连接事件
	 * @param $server
	 * @param $fd
	 */
	public function onClose($server, $fd){
		echo "关闭TCP 连接".PHP_EOL;
	}

	/**
	 * 发起Get 或 Post 请求
	 * @param string $url           请求地址
	 * @param array $request_data   请求参数
	 * @param string $request_type  请求类型
	 * @param array $headers        头信息
	 * @param bool $is_ssl          是否是ssl
	 * @return bool|string
	 */
	public function curl($url = '', $request_data = [], $request_type = 'get', $headers = [], $is_ssl = false)
	{
		$curl = curl_init (); // 初始化
		// 设置 URL
		curl_setopt($curl, CURLOPT_URL, $url);
		// 不返回 Response 头部信息
		curl_setopt ( $curl, CURLOPT_HEADER, 0 );
		// 如果成功只将结果返回，不自动输出任何内容
		curl_setopt ( $curl, CURLOPT_RETURNTRANSFER, 1 );
		// 设置请求参数
		curl_setopt ( $curl, CURLOPT_POSTFIELDS, http_build_query($request_data));
		// TRUE 时追踪句柄的请求字符串
		curl_setopt($curl, CURLINFO_HEADER_OUT, true);
		// Post 类型增加以下处理
		if( $request_type == 'post') {
			// 设置为POST方式
			curl_setopt ( $curl, CURLOPT_POST, 1 );
			// 设置头信息
			curl_setopt($curl, CURLOPT_HTTPHEADER, array('Content-Type: application/json', 'Content-Length:' . strlen(json_encode($request_data))));
			// 设置请求参数
			curl_setopt ( $curl, CURLOPT_POSTFIELDS, json_encode($request_data));
			// 当POST 数据大于1024 时强制执行
			curl_setopt ( $curl, CURLOPT_HTTPHEADER, array("Expect:"));
		}

		// 判断是否绕过证书
		if( $is_ssl ) {
			//绕过ssl验证
			curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
			curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
		}
		if(!empty($headers))  curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
		// 执行
		$result = curl_exec ( $curl );
		if ( $result == FALSE) return false;
		// 关闭资源
		curl_close ( $curl );
		return $result;
	}

	/**
	 * 写入日志
	 * @param $content
	 */
	public function addLog($content){
		$path = dirname(__FILE__)."/logs/";
		if (!is_dir($path))
			mkdir($path,0777,true);

		$file_name = $path.date("Y_m_d") . ".log";
		if (!file_exists($file_name)) {
			touch($file_name);
			chown($file_name, 0777);
		}

		$file_log = fopen($file_name, "a");
		fputs($file_log, $content);
		fclose($file_log);
	}
}

$server = new taskServer();
```

### 客户端
这里的客户端可以是 cli 脚本，也可以是对应控制器中的具体方法，只要能连接Swoole 监听的Tcp 就行。

```
<?php
namespace app\admin\controller;

class Index extends Base
{
    public function index(){
        $client = new \Swoole\Client(SWOOLE_SOCK_TCP);
  	    if (!$client->connect('0.0.0.0', 9501)) {
  		    return json("connect failed. Error: {$client->errCode}\n");
  	    }
  	    $data = [
  			"url" => "https://api.paasoo.com/json",
  			"data" => [
  				"key" => "key",
  				"secret" => "secret",
  				"from" => "sms",
  				"to" => "mobile_phone",
  				"text" => "test",
  			],
  			"type" => "get"
  		];
	    $client->send(json_encode($data));
	    return json($client->recv());
    }
}
```

### 参考链接
* [php使用Swoole来实现实时异步任务队列](https://www.phpernote.com/php-function/1450.html)
* [基于 swoole 下 异步消息队列 API](https://learnku.com/articles/43752)