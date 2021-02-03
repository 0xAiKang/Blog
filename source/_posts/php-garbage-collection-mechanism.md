---
title: PHP 垃圾回收机制
date: 2021-01-04 22:44:06
tags: ["PHP"]
categories: ["PHP"]
---

PHP 是一门托管型语言，在PHP 编程时，程序员不需要手动处理内存资源的分配和释放，这就意味着 PHP 本身实现了垃圾回收机制（Garbage Collection）。

<!-- more -->

> 垃圾回收机制是什么？

垃圾回收是一种自动的存储器管理机制，当某个程序占用的一部分内存空间不再被这个程序所访问时，这个程序会借助垃圾回收算法自动向操作系统归还这部分的内存空间。

## PHP 引用计数
定义一个PHP 变量如下：
```
<?php
$str = "boo";
$str_bak = $var;
unset($str);
```
上面这几行代码分别做了如下事情：
1. 第一行代码创建了一个字符串变量，申请了一个大小为 三个字节的内存空间，保存了字符串 `boo` 和一个 `NULL(\0)`的结尾。
2. 第二行代码定义了一个新的字符串变量，并将变量`str`的值复制给了这个新的变量。
3. 第三行 `unset` 掉了变量 `str`

这样的代码在很常见，如果PHP 对于每一个变量赋值都重新分配内存，copy 数据的话，那么上面的那段代码就需要共申请六个字节的内存空间，而我们也很容易看出来，其实完全没有必要申请两份空间。

PHP中的变量是用一个存储在 `symbol_table` 中的符号名，对应一个 `zval` 变量容器来实现的，比如对于上面的第一行代码，会在 `symbol_table` 中存储一个值 `str`, 对应的有一个指针指向一个 `zval `结构，变量值 `boo` 保存在这个变量容器中，所以不难想象，对于上面的代码来说，我们完全可以让 `str` 和 `str_bak` 对应的指针都指向同一个变量容器就可以了。

PHP 也是这样做的，这是就需要介绍 `zval`变量容器的结构了。

每个变量存在一个叫做`zval`的变量容器中。一个`zval`变量容器，除了包含变量的类型和值，还包括两个字节的额外信息：
1. `is_ref`：bool 值，用来标示这个变量是否属于引用集合（reference_set）。通过这个字节，PHP 引擎才能把普通变量和引用变量区分开来。
2. `refcount`：用以表示指向这个 `zval` 变量容器的变量个数。

### 1. 查看内部结构
当一个变量被赋值时，就会生成一个`zval`变量容器：
```
<?php
$str = "hello, php";
xdebug_debug_zval('str');
```
在 PHP 中可以通过 xdebug 扩展中提供的方法`xdebug_debug_zval()`来查看变量的计数变化。

输出结果
```
str:(refcount=1, is_ref=0)string 'hello, php' (length=10)
```

### 2. 增加引用次数
把一个变量赋值给另一个变量将增加引用次数（refcount + 1）：
```
$str = "hello, php";
$str2 = $str
xdebug_debug_zval('str');
```

输出结果：
```
str:(refcount=2, is_ref=0)string 'hello, php' (length=10)
```

这时，引用次数是 `2`，这是因为同一个变量容器被变量a 和变量b 关联，当任何关联到的某个变量容器离开它的作用域（比如：函数执行结束），或者对变量调用了 `unset()` 函数，`refcount`的值就会 `-1`，当没必要时，PHP 不会再去复制已生成的变量容器，变量容器在`refcount`的值变为 0 时，就会被销毁。

### 3. 数组型的变量

```
<?php
$arr = ['a'=>'hello', 'b'=>'php'];
xdebug_debug_zval('arr');
```

输出结果：
```
arr:
(refcount=2, is_ref=0)
array (size=2)
  'a' => (refcount=1, is_ref=0)string 'hello' (length=5)
  'b' => (refcount=1, is_ref=0)string 'php' (length=3)
```

### 4. 引用赋值
```
<?php
$str = "hello, php";
$str_bak = &$str;
xdebug_debug_zval('str');
```

输出结果：
```
str:(refcount=2, is_ref=1)string 'hello, php' (length=10)
```
`is_ref = 1`表示被引用次数为 `1`。

### 5. 销毁变量

```
<?php
$a = "new string";
$c = $b = $a;
xdebug_debug_zval( 'a' );
unset( $b, $c );
xdebug_debug_zval( 'a' );
unset( $a);
xdebug_debug_zval( 'a' );
```

输出结果：
```
a:(refcount=3, is_ref=0)string 'new string' (length=10)
a:(refcount=1, is_ref=0)string 'new string' (length=10)
a: no such symbol
```
可以看到当销毁变量a之后，与之包含类型的值和变量容器就会从内存中删除。

## 测试垃圾回收机制

下面用一个比较经典的内存泄露例子来测试垃圾回收机制，通过创建一个对象，这个对象中的一个属性被设置为对象本身，在下一个循环（iteration）中，当脚本中的变量被重新赋值时，就会发生内存泄漏。
```
<?php
class Foo
{
    public $var = '3.1415962654';
}

for ( $i = 0; $i <= 1000000; $i++ )
{
    $a = new Foo;
    $a->self = $a;
}

echo memory_get_peak_usage(), "\n";
```

以我本地的机器为例，分别在打开/关闭垃圾回收机制（通过配置 zend.enable_gc实现）的情况下运行脚本，并记录时间。
```
$ time php -dzend.enable_gc=0 -dmemory_limit=-1 -n get_memory.php
440776744
php -dzend.enable_gc=0 -dmemory_limit=-1 -n   0.22s user 0.23s system 39% cpu 1.145 total

$ time php -dzend.enable_gc=1 -dmemory_limit=-1 -n get_memory.php
4839240
php -dzend.enable_gc=1 -dmemory_limit=-1 -n   0.42s user 0.03s system 76% cpu 0.588 total
```

这个测试并不能代表真实应用程序的情况，但是它的确显示了新的垃圾回收机制在内存占用方面的好处。而且在执行中出现更多的循环引用变量时，内存节省会更多。

## 垃圾回收相关配置
可以通过修改配置文件 `php.ini` 中的 `zend.enable_gc` 来打开或关闭 PHP 的垃圾回收机制。

> 刚好借着PHP 的垃圾回收这个主题解释一个问题：PHP 是否可以常驻内存？

答案是：传统的PHP 无法以常驻内存的方式运行。

这是因为PHP 是一种解释型脚本语言，这种运行机制使得每个PHP 页面解释执行完之后，所有资源都被回收掉了。

不过好在Swoole 的出现为PHP 弥补了这一缺陷（这里用缺陷这个词并不合适，毕竟每一种语言工具应该尽可能扬长避短）。

### 参考链接
* [PHP二十一问：PHP的垃圾回收机制](https://www.iminho.me/wiki/blog-18.html)
* [引用计数基本知识解释垃圾回收机制](https://www.php.net/manual/zh/features.gc.refcounting-basics.php)