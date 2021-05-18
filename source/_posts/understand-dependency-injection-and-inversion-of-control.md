---
title: 用一个 IoC 容器来理解什么是依赖注入/控制反转
date: 2021-05-17 21:03:14
tags: ["PHP"]
cagegories: ["PHP"]
---

经常会听到『依赖注入』(DependencyInjection)和『控制反转』(Inversion of Control)这两个名词，初学者往往会被其给吓住，误以为是什么特别高深的技术，其实了解了来龙去脉之后，就会发现仅仅只是名字听上去高大上而已。

<!-- more -->

## 依赖注入
依赖和注入其实说的是同一个东西，它们只是一种编程的思想，其主要作用是用于减少程序间的耦合。以及有效分离对象和它所需的外部资源。

下面先来看一个简单的小例子来体会下什么是『依赖注入』：

`car.php`
```php
<?php
namespace di;

class Car
{
    public function pay()
    {
        return "199";
    }
}
```

`person.php`
```php
<?php
namespace di;

class Person
{
    public function buy()
    {
        $bwm = new Car();
        return $bwm->pay();
    }
}
```

`index.php`
```php
<?php
$boo = new \di\Person();

echo $boo->buy();  // 199
```

在上面这个例子（主要看Person 这个类），需要明确几个概念：
1. 依赖：谁依赖了谁？
2. 注入：谁又注入了谁？

通过观察可以发现：
1. Car 类在Person 类中实例化（Person 类『依赖』于Car 类）
2. 但此时并没有发现谁『注入』谁

观察`buy()` 这个方法，假如需求发生变化，需要买的不是一辆车，而是一栋房，那么还得更改 Person 类的源码，由实例化一辆车改为实例化一栋房。更好的做法应该是，把所需的实例，通过函数参数的方式传入进来。

`Person.php`
```php
<?php
namespace di;

class Person
{
    public function buy($obj)
    {
        return $obj->pay();
    }
}
```

`index.php`：
```php
<?php
$boo = new \di\Person();

$car = new \di\Car();
echo $boo->buy($car);  // 199
```

现在可以清晰的看到，`Car类`通过函数参数的方式『注入』到了`Person 类`中。

『依赖注入』有多种，这里只是用一个最简单的例子来解释『依赖注入』的思想以及可以有效地解决什么问题：
1. 减少程序间的耦合
2. 分离对象和它所需的外部资源

## 控制反转

那么啥是控制反转呢？

再次观察最开始的代码，可以发现，Car 类在Person 类中是**被动实例化**，Person 类**正向控制**了Car 类，其实例化顺序是先有了Person 类才有Car 类：`Person -> Car`。

改成后面的注入方式之后，则可以发现，Car 类是**主动实例化**，Person 类失去了对Car 类的控制权，其实例化顺序是先有Car 类才有Person 类：`Car -> Person`。

这就是『控制反转』。

其实很多时候，我们在不经意间都有使用到这种思想，只是自己没有意识到。

## IoC 容器
Ioc容器（Inversion of Control）常常伴随着依赖注入、控制反转一起出现，那么它倒底是个什么东西呢？

在回答这个问题之前，先来看看上面的那段代码，虽然最后使用依赖注入的方式解耦了`Person类` 和`Car 类`，但此时又会面临一个新的问题：依赖仍然需要手动创建，此时只有两个类相互依赖还好，一旦类的依赖关系，嵌套过深，手动创建就会变成一件麻烦事：

```php
$boo = new \di\Person();
echo $boo->buy(new \di\Car());  // 199
```

这时候，就需要IoC 容器登场了。

IoC 容器的核心是通过PHP 的 [反射 (Reflection)](https://www.php.net/manual/en/book.reflection.php) 来实现的。

`IoC.php`：
```php
<?php
namespace di;

class IoC
{
    public static function maket($className)
    {
        $reflect = new \ReflectionClass($className);
        $construct = $reflect->getConstructor();
        if (!$construct) {
            return new $className;
        }

        $params = $construct->getParameters();
        if (empty($params)) {
            return new $className;
        }

        $args = [];
        foreach ($params as $param) {
            $class = $param->getClass();
            if ($class) {
                $args[] = static::make($class->name);
            }
        }

        return $reflect->newInstanceArgs($args);
    }
}
```

`person.php`
```php
<?php
namespace di;

class person
{
    protected $obj;

    /**
     * 需要注意：要想反射能够识别，此处必须给参数声明类型。
     */
    public function __construct(car $obj)
    {
        $this->obj = $obj;
    }

    public function buy()
    {
        return $this->obj->pay();
    }
}
```

`index.php`
```php
<?php
$boo = \di\IoC::make("\di\person");
echo $boo->buy(); // 199
```

可以看到，即使没有手动创建Car 类，也不会影响Person 类的调用，这就是IoC 容器所解决的问题：把对象与对象之间的依赖关系隐藏到容器（存储实例化对象）中，通过自动创建的方式解决依赖关系。

这里仅仅只是抛砖引玉，用一个简单的IoC 容器例子来理解什么是依赖注入/控制反转，更多相关知识可以通过查看主流框架源码或者[PHP-DI](https://php-di.org) 进行学习。

## 参考链接
* [写一个简单的IoC容器案例，理解什么是依赖注入和控制反转](https://learnku.com/articles/56111)