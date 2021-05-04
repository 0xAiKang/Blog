---
title: Laravel 中 Collection 的实际使用
date: 2021-05-04 08:32:36
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

这篇笔记用来整理Collection 在Laravel 的实际应用场景。

<!-- more -->

## 求和
> 需求：遍历$orders 数组，求price 的和。

```php
<?php
// 引入package
require __DIR__ . '/vendor/autoload.php';

$orders = [[
    'id'            =>      1,
    'user_id'       =>      1,
    'number'        =>      '13908080808',
    'status'        =>      0,
    'fee'           =>      10,
    'discount'      =>      44,
    'order_products'=> [
        ['order_id'=>1,'product_id'=>1,'param'=>'6寸','price'=>555.00,'product'=>['id'=>1,'name'=>'蛋糕名称','images'=>[]]],
        ['order_id'=>1,'product_id'=>1,'param'=>'7寸','price'=>333.00,'product'=>['id'=>1,'name'=>'蛋糕名称','images'=>[]]],
    ],
]];
```

1. 使用传统的foreach 方式进行遍历：
```php
$sum = 0;
foreach ($orders as $order) {
    foreach ($order['order_products'] as $item) {
        $sum += $item['price'];
    }
}
echo $sum;
```

2. 使用集合的map、flatten、sum：
```php
$sum = collect($orders)->map(function($order){
    return $order['order_products'];
})->flatten(1)->map(function($order){
    return $order['price'];
})->sum();

echo $sum;
```

map：遍历集合，返回一个新的集合。
flatten：将多维数组转换为一维。
sum：返回数组的和。

3. 使用集合的flatMap、pluck、sum：
```php
$sum = collect($orders)->flatMap(function($order){
    return $order['order_products'];
})->pluck('price')->sum();
echo $sum;
```

flatMap：和`map` 类似，不过区别在于`flatMap` 可以直接使用返回的新集合。

4. 使用集合的flatMap、sum：
```php
$sum = collect($orders)->flatMap(function($order){
    return $order['order_products'];
})->sum('price');
```

sum：可以接收一个列名作为参数进行求和。

## 格式化数据
> 需求：将如下结构的数组，格式化成下面的新数组。

```php
// 带格式化数组
$gates = [
    'BaiYun_A_A17',
    'BeiJing_J7',
    'ShuangLiu_K203',
    'HongQiao_A157',
    'A2',
    'BaiYun_B_B230'
];

// 新数组
$boards = [
    'A17',
    'J7',
    'K203',
    'A157',
    'A2',
    'B230'
];
```

1. 使用foreach 进行遍历：
```php
$res = [];
foreach($gates as $key => $gate) {
    if(strpos($gate, '_') === false) {
        $res[$key] = $gate;
    }else{
        $offset = strrpos($gate, '_') + 1;
        $res[$key] = mb_substr($gate , $offset);
    }
}
var_dump($res);
```

2. 使用集合的map以及php 的explode、end：
```php
$res = collect($gates)->map(function($gate) {
    $parts = explode('_', $gate);
    return end($parts);
});
```

3. 使用集合的map、explode、last、toArray：
```php
$res = collect($gates)->map(function($gate) {
    return collect(explode('_', $gate))->last();
})->toArray();
```

explode：将字符串进行分割成数组
last：获取最后一个元素

## 统计GitHub Event
首先，通过此[链接](https://api.github.com/users/YOUR_USRE_NAME/events)获取到个人事件json。

一个 `PushEvent计` 5 分，一个 `CreateEvent` 计 4 分，一个 `IssueCommentEvent计` 3 分，一个 `IssueCommentEvent` 计 2 分，除此之外的其它类型的事件计 1 分，计算当前用户的时间得分总和。
```php
$opts = [
        'http' => [
                'method' => 'GET',
                'header' => [
                        'User-Agent: PHP'
                ]
        ]
];
$context = stream_context_create($opts);
$events = json_decode(file_get_contents('http://api.github.com/users/0xAiKang/events', false, $context), true);
```

1. 传统foreach 方式：

```php
$eventTypes = []; // 事件类型
$score = 0; // 总得分
foreach ($events as $event) {
    $eventTypes[] = $event['type'];
}

foreach($eventTypes as $eventType) {
    switch ($eventType) {
        case 'PushEvent':
        $score += 5;
        break;
        case 'CreateEvent':
        $score += 4;
        break;
        case 'IssueEvent':
        $score += 3;
        break;
        case 'IssueCommentEvent':
        $score += 2;
        break;
        default:
        $score += 1;
        break;
    }
}
```

1. 使用集合的map、pluck、sum 方法：
```php
$score = $events->pluck('type')->map(function($eventType) {
   switch ($eventType) {
      case 'PushEvent':
      return 5;
      case 'CreateEvent':
      return 4;
      case 'IssueEvent':
      return 3;
      case 'IssueCommentEvent':
      return 2;
      default:
      return 1;
  }
})->sum();
```
使用集合的链式编程，可以很好地解决上面那种多次遍历的问题。

2. 使用集合中的map、pluck、get 方法：
```php
$score = $events->pluck('type')->map(function($eventType) {
   return collect([
       'PushEvent'=> 5,
       'CreateEvent'=> 4,
       'IssueEvent'=> 3,
       'IssueCommentEvent'=> 2
   ])->get($eventType, 1); // 如果不存在则默认等于1
})->sum();
```

3. 尝试将该需求，封装成一个类：
```php
class GithubScore {
    private $events;

    private function __construct($events){
        $this->events = $events;
    }

    public static function score($events) {
        return (new static($events))->scoreEvents();
    }

    private function scoreEvents() {
        return $this->events->pluck('type')->map(function($eventType){
            return $this->lookupEventScore($eventType, 1);
        })->sum();
    }

    public function lookupEventScore($eventType, $default_value) {
       return collect([
           'PushEvent'=> 5,
           'CreateEvent'=> 4,
           'IssueEvent'=> 3,
           'IssueCommentEvent'=> 2
       ])->get($eventType, $default_value); // 如果不存在则默认等于1
    }
}

var_dump(GithubScore::score($events));
```

## 格式化数据
> 需求：将以下数据格式化成新的结构。

```php
$messages = [
    'Should be working now for all Providers.',
    'If you see one where spaces are in the title let me know.',
    'But there should not have blank in the key of config or .env file.'
];

// 格式化之后的结果
- Should be working now for all Providers. \n
- If you see one where spaces are in the title let me know. \n
- But there should not have blank in the key of config or .env file.
```

1. 传统的foreach 方式：
```php
$comment = '- ' . array_shift($messages);
foreach ($messages as $message) {
    $comment .= "\n -  ${message}";
}
var_dump($comment);
```

2. 使用集合的map、implode方法：
```php
$comment = collect($messages)->map(function($message){
    return '- ' . $message;
})->implode("\n");
var_dump($comment);
```

## 多个数组求差
> 需求：两组数据分别代表去年的营收和今年的营收，求每个月的盈亏情况。

```php
$lastYear = [
    6345.75,
    9839.45,
    7134.60,
    9479.50,
    9928.0,
    8652.00,
    7658.40,
    10245.40,
    7889.40,
    3892.40,
    3638.40,
    2339.40
];

$thisYear = [
    6145.75,
    6895.00,
    3434.00,
    9349350,
    9478.60,
    7652.80,
    4758.40,
    10945.40,
    3689.40,
    8992.40,
    7588.40,
    2239.40
];
```

1. 传统的foreach 方式：
```php
$profit = [];
foreach($thisYear as $key => $monthly){
    $profit[$key] = $monthly - $lastYear[$key];
}
var_dump($profit);
```

2. 使用集合的zip、first、last：
```php
$profit = collect($thisYear)->zip($lastYear)->map(function($monthly){
    return $monthly->first() - $monthly->last();
});
```
zip：将给定数组的值与相应索引处的原集合的值合并在一起。

## 创建lookup 数组
> 需求：将如下数组格式化成下面的结果：

```php
$employees = [
    [
        'name' => 'example',
        'email' => 'example@exmaple.com',
        'company' => 'example Inc.'
    ],
    [
        'name' => 'Lucy',
        'email' => 'lucy@example.com',
        'company' => 'ibm Inc.'
    ],
    [
        'name' => 'Taylor',
        'email' => 'toylor@laravel.com',
        'company'=>'Laravel Inc.'
    ]
];

// 格式化之后的结果
$lookup = [
    'example' => 'example@example.com',
    'Lucy' => ‘lucy@example.com’,
    'Taylor'=> 'toylor@laravel.com'
];
```

1. 传统的foreach 方式：
```php
$emails = [];
foreach ($employees as $key => $value) {
    $emails[$value['name']] = $value['email'];
}
```

2. 使用集合的reduce 方法：
```php
$emails = collect($employees)->reduce(function($emailLookup, $employee){
    $emailLookup[$employee['name']] = $employee['email'];
    return $emailLookup;
},[]);
```

reduce：将每次迭代的结果传递给下一次迭代直到集合减少为单个值。

3. 使用集合的pluck 方法：
```php
$emails = collect($employees)->pluck('name', 'email');
```