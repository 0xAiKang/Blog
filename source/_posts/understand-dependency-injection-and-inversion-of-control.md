---
title: 理解依赖注入和控制反转
date: 2021-05-17 21:03:14
tags: ["PHP"]
cagegories: ["PHP"]
---

经常会听到『依赖注入』(DependencyInjection)和『控制反转』(Inversion of Control)这两个名词，初学者往往会被其给吓住，误以为是什么特别高深的技术，其实了解了来龙去脉之后，就会发现仅仅只是名字听上去高大上而已。

<!-- more -->

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

那么啥是控制反转呢？

再次观察最开始的代码，可以发现，Car 类在Person 类中是**被动实例化**，Person 类**正向控制**了Car 类，其实例化顺序是先有了Person 类才有Car 类：`Person -> Car`。

改成后面的注入方式之后，则可以发现，Car 类是**主动实例化**，Person 类失去了对Car 类的控制权，其实例化顺序是先有Car 类才有Person 类：`Car -> Person`。

这就是『控制反转』。

其实很多时候，我们在不经意间都有使用到这种思想，只是自己没有意识到。下一篇笔记中，通过一个小案例来理解什么是`IoC` 容器。