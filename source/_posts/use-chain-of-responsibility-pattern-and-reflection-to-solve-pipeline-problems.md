---
title: 使用职责链模式和反射解决流水线问题
date: 2021-05-15 22:19:28
tags: ["PHP", "Laravel", "设计模式"]
categories: ["PHP", "Laravel", "设计模式"]
---

~~设计模式是区分程序员能力大小的一个重要因素——Boo。~~

<!-- more -->

## 需求分析
原来的业务可以看成是一条流水线，这条流水线上有各个模块有各自的职责，相互依赖但并不耦合，操作A 完成之后，才能执行操作B，操作完成之后才能执行操作C... 以此类推。

就目前的需求来看，直接使用传统的方式进行编码，各个职责所对应的功能直接写到控制器中，即可。但问题在于，同时有多个不同的角色，可能会调用该功能，比如：

> 用户只能执行A、C 操作，管理员只能执行A、B、C 操作，而超级管理员则可以执行所有操作。

如果仍然坚持使用传统的方式进行编码，那么同一个操作，可能需要在不同角色模块下各自维护一份，一旦其中某一个的需求发生了变化，那么还得同时更正好几份代码...

已知需求：
* 每个角色所需要执行的操作内容都是一样的，并不会是因为身份是管理员或者用户，其操作就会发生变化。
* 所有的操作在逻辑上相互依赖，但并不耦合。
* 每个角色所能执行的操作是已知的。
* 禁止越级操作，不能直接越过A 去执行B 操作。

## 责任链模式
基于以上几点，最终选择『责任链模式』作为设计思路，原因有以下：
1. 职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解耦了。
2. 在没达到指定条件前，会一直向下传递，直到结束流水线

## 实现

伪代码分析过程：
1. 新建一个订单操作类，在构造函数中分别接收用户身份和订单ID，根据当前订单状态及用户身份获取各自当前所能执行的操作（这里有个问题，如果角色增加，构造函数会因此变得复杂，可以配合使用其他设计模式代替在构造函数中赋值）。
2. 在该类中配合PHPStorm 注解定义所有需要执行的操作
3. 通过魔术方式`__call` 进行反射，将操作映射到具体功能实现的类中

核心有两点：
1. 通过职责连获取不同角色所能执行的操作
2. PHPStorm 注解配合反射使用，查找并执行具体功能

### 步骤一
创建职责连：

用户：
```php
<?php 
namespace App\Services\order\action;

class UserOrderActionService
{
    private $order;
    
    public function __construct($order)
    {
        $this->order = $order;
    }

    static public function getAction($order)
    {
        return (new self($order))->step1();
    }
    
    private function step1()
    {
        if ($this->order->order_type == OrderType::SUCCESS) {
            return [
                'actionA',
            ];
        }
        return $this->step2();
    }
    
    private function step2()
    {
        if ($this->order->order_type == OrderType::FAIL) {
            return [
                'actionD',
            ];
        }
        return $this->step3();
    }
    
    //...
    
    private function stepEnd()
    {
        return [];
    }
}
```

管理员：
```php
<?php 
namespace App\Services\order\action;

class AdminOrderActionService
{
    private $order;
    
    public function __construct($order)
    {
        $this->order = $order;
    }

    static public function getAction($order)
    {
        return (new self($order))->step1();
    }
    
    private function step1()
    {
        if ($this->order->order_type == OrderType::SUCCESS) {
            return [
                'actionA',
                'actionB',
                'actionC',
            ];
        }
        return $this->step2();
    }
    
    private function step2()
    {
        if ($this->order->order_type == OrderType::FAIL) {
            return [
                'actionD',
            ];
        }
        return $this->step3();
    }
    
    //...
    
    private function stepEnd()
    {
        return [];
    }
}
```

超级管理员：
```php
<?php
namespace App\Services\order\action;

class SuperAdminOrderActionService
{
    private $order;
    
    public function __construct($order)
    {
        $this->order = $order;
    }

    static public function getAction($order)
    {
        return (new self($order))->step1();
    }
    
    private function step1()
    {
        if ($this->order->order_type == OrderType::SUCCESS) {
            return [
                'actionA',
                'actionB',
                'actionC',
            ];
        }
        return $this->step2();
    }
    
    private function step2()
    {
        if ($this->order->order_type == OrderType::FAIL) {
            return [
                'actionC',
            ];
        }
        return $this->step3();
    }
    
    //...
    
    private function stepEnd()
    {
        return [];
    }
}
```

### 步骤二

虽然不同的操作操作之间并没有直接关联，此处为了方便日后功能扩展，还是将不同类型的操作进行了分类

OrderSuccessService：
```php
<?php

namespace App\Services\order;

class OrderSuccessService 
{
    // ...
  
    public function actionA(array $param){
       try{
          // 具体业务逻辑
          
       }catch(OrderException $exception){
          throw new \Exception($exception->getMessage());
       }
    }
    
    public function actionB(array $param){
        // ...
    }
    
    public function actionC(array $param){
        // ...
    }
    
    // ...
}
```

OrderFailService：
```php
<?php

namespace App\Services\order;

class OrderFailService 
{
    // ...
  
    public function actionD(array $param){
       try{
          // 具体业务逻辑
          
       }catch(OrderException $exception){
          throw new \Exception($exception->getMessage());
       }
    }
    
    // ...  
}
```

### 步骤二
创建OrderAction 类：

```php
<?php
namespace App\Services\order\action;

use App\Models\OrderModel;
use App\Services\order\action\UserOrderActionService;
use App\Services\order\action\AdminOrderActionService;
use App\Services\order\action\SuperAdminOrderActionService;

/**
 * 注解部分 
 * @method OrderSuccessService actionA()                操作A
 * @method OrderSuccessService actionB()                操作B
 * @method OrderSuccessService actionC()                操作C
 * @method OrderFailService actionD()                   操作D
 * // ...
 * Class OrderAction
 * @package App\Services\order\action
 */
class OrderAction
{
  private $actionCode;
  
  private $class = [
      "App\Services\order\OrderSuccessService",
      "App\Services\order\OrderFailService",
      // ...
  ];

  public function __construct(string $role_type, int $order_id)
    {
        $order = OrderModel::find($order_id);

        switch ($role_type) {
            case "user":
                $this->actionCode = UserOrderActionService::getAction($order);
                break;
            
            case "admin":
                $this->actionCode = AdminOrderActionService::getAction($order);
                break;
              
            case "superAdmin":
                $this->actionCode = superAdminOrderActionService::getAction($order);
                break;
                
            default :
                // ... 
        }
    }
    
    private function strategy($name)
    {
        foreach ($this->class as $class) {
            if (in_array($name, get_class_methods($class))) {
                return new $class($this->actionUser);
            }
        }
        return $this;
    }
    
    private function auth(string $method)
    {
        if (!in_array($method, $this->actionCode)) {
            throw new \Exception("非法操作，禁止越级操作");
        }
        return $this;
    }
    
    public function __call($name, $arguments)
    {
        return $this->auth($name)->strategy($name)->$name(...$arguments);
    }
}
```

### 步骤四
控制器调用：

```php
namespace App\Services\user;

class OrderController
{
   public function actionA(OrderRequest $request)
   {
      $param = $request->only(['order_id']);
   
      try {
          (new OrderAction("worker", $param['order_id']))->actionA($param);
      } catch (OrderException $exception) {
          return $this->failed($exception->getMessage());
      }
      
      return $this->success();
   }
}
```

至此，该业务的核心流程已经通过职责连模式和反射实现。

回头再看看这些代码，其实也不会有什么难度，只是自己在这方便的锻炼太少了，每每遇到问题，总是很难将需求抽象，或者尽管知道用什么设计模式，但最终导致写不出来。