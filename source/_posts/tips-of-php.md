---
title: Tips of PHP
date: 2021-04-28 22:28:49
tags: ["PHP"]
categories: ["PHP"]
---

Awesome tips for PHP.

<!-- more -->

## 减少if...else 的使用
`if...else` 通常是一个糟糕的选择，它导致设计复杂，代码可读性差，并且可能导致重构困难。

```php
function doSomething() {
    if (...) {
        if (...) {
            // ...
        } esle {
            // ...
        }
    } else {
        if (...) {
            // ...
        } esle {
            // ...
        }
    }
}
```

好在可以通过其他方式可以避免对`if...else` 的过度依赖：

1. 提前 return：
```php
public function run($input){
  if ($input > 5){
    // do something 
  } else {
    // do something else
  }
}
```

只需要删除`else` 块，即可简化此过程：
```php
public function run($input){
  if ($input > 5){
    // do something 
    return;
  }
  
  // do something else 
}
```

2. `switch...case` 也是不错地选择：
```php
public function run($input){
  switch($input){
    case "A":
      // do something
      break;
      
    case "B":
      // do else something
      break
      
    // ...
  }
}
```

## 使用try...catch

```php
class UserModel
{
    public function login($username = '', $password = '')
    {
        if (...) {
            // 用户不存在
            return -1;
        }
        
        if (...) {
            // 密码错误
            return -2;
        }
        
        // code...
    }
}

class UserController
{
    public function login($username = '', $password = '')
    {
        $model = new UserModel();
        $res   = $model->login($username, $password);
        
        if ($res === -1) {
            return [
                'code' => '404',
                'message' => '用户不存在'
            ];
        }
        
        if ($res === -2) {
            return [
                'code' => '400',
                'message' => '密码错误'
            ];
        }
        
        // code...
    }
}
```

使用`try...catch` 重写：
```php
class UserModel
{
    public function login($username = '', $password = '')
    {
        if (...) {
            // 用户不存在
            throw new Exception('用户不存在', '404');
        }

        if (...) {
            // 密码错误
            throw new Exception('密码错误', '400');
        }
        
        // ...
    }
}

class UserController
{
    public function login($username = '', $password = '')
    {
        try {
            $model = new UserModel();
            $res   = $model->login($username, $password);
            
            // 如果需要的话，我们可以在这里统一commit数据库事务
            // $db->commit();
        } catch (Exception $e) {
        
            // 如果需要的话，我们可以在这里统一rollback数据库事务
            // $db->rollback();
        }
        
        // ...
    }
}
```
通过使用`try...catch` 重写，使得代码逻辑职责分明、更加清晰。`try` 只用关心业务正常情况的处理，而所有异常则统一在`catch` 中处理，上游只需将异常抛出即可。

## 使用匿名函数
需要在一个方法中，重复处理某个逻辑，这时可能会将其封装成一个函数，即：

```php
function doSomething(...) {
    // ...
    format(...);
    
    // ...
    format(...);
    
    // ...
}

// 声明一个格式化代码的函数或方法
function format() {
    // 格式化代码段
    // ...
}
```

上面这段代码并没有什么问题，但是如果这个函数仅仅只在`doSomething` 中使用呢？更好地做法应该是这样：
```php
function doSomething() {
    // ...
    $format = function (...) use (...) {
        // 格式化代码段
        // ...
    };
    
    // ...
    $format(...);
    
    // ...
    $format(...);
    
    //...
}
```

## 简单的策略模式

客户端在做决策时，通常会根据不同的上下文环境选择不同的策略，可能会写成下面这样：

```php
class One
{
    public function doSomething()
    {
        if (...) {
            $instance = new A();
        }　elseif (...) {
            $instance = new B();
        } else {
            $instance = new C();
        }
        
        $instance->doSomething(...);
        // ...
    }
}
```

上面这种情况，无论是使用`if...else`还是`switch...case` 当策略增多时，都会出现大量分支逻辑判断，好写的做法是定义一个简单的策略：
```php
class One
{
    private $map = [
        'a' => 'namespace\A',
        'b' => 'namespace\B',
        'c' => 'namespace\C'
    ];
    
    public function doSomething()
    {
        // ...
        $instance = new $this->map[$strategy];
        $instance->doSomething(...);
        
        // ...
    }
}
```

## 依赖注入

```php
class One
{
    private $instance;

    public function __construct()
    {  
        $this->intance = new Two();
    }

    public function doSomething()
    {
        if (...) {
            // 如果某种情况调用类Two的实例方法
            $this->instance->do(...);
        }
        ...
    }
}

$instance = new One();
$instance->doSomething();
```
上面这段代码有什么问题？
1. 不符合设计模式的最少知道原则，One 类内部直接依赖了Two 类。


使用依赖注入重写此类：
```php
class One
{
    private $closure;

    public function __construct(Closure $closure)
    {  
        $this->closure = $closure;
    }

    public function doSomething()
    {
        if (...) {
            // 用的时候再实例化
            // 实现懒加载
            $instance = $this->closure();
            $instance->do(...)
        }
        ...
    }
}
...

$instance = new One(function () {
    // 从外部注入Two 类
    return new Two();
});
$instance->doSomething();
```

## 原文地址
* [easy-tips/php](https://github.com/TIGERB/easy-tips/blob/master/php/artisan.md)