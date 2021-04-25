---
title: Laravel 中的 Collection 基本使用
date: 2021-04-25 23:17:05
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

集合是Laravel 中提供的最强大的功能之一，集合本质上是由功能强大的数组组成。

<!-- more -->

把类似数组的对象应用到方法中是很有用的，通过链式编程，用极短的代码，就可以达到预期的效果。

需要注意的是集合并不是Laravel 中独有的，许多语言都可以在数组中使用集合式编程，但非常遗憾，原生的PHP 是不支持集合式编程的，不过幸运的是，一些勤劳的人已经为我们完成了艰苦的工作，并编写了一个非常方便的包——[illuminate/support](https://github.com/illuminate/support)、[Tightenco/Collect](https://github.com/tighten/collect) 。

一般来说，集合是不可改变的，这意味着每个 Collection 方法都会返回一个全新的 Collection 实例。

## 创建集合

为了创建一个集合，可以将一个数组传入集合的构造器中，也可以创建一个空的集合，然后把条目写到集合中。Laravel 也有`collect()`助手，这是最简单的，新建集合的方法。

```php
$collection = collect([1, 2, 3]);
```

> 默认情况下， Eloquent 查询的结果返回的内容都是 `Illuminate\Support\Collection` 实例，如果希望对结果进行序列化，可以使用`toArray()`、`toJson()` 方法。

记住，所有方法都可以使用链式编程的方式优雅的操纵数组。而且，几乎所有的方法都会返回**新的** `Collection` 实例，

## all
返回该集合表示的底层数组。

```php
collect(["boo", "yumi", "mac"])->all();
// [“boo”, "yumi", "mac"]
```

## avg
获取数组的平均值：
```php
collect([1, 1, 2, 4])->avg(); // 2
```

获取二维数组的平均值：
```php
collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo'); // 20
```
`avg()`是`average()` 的别名，两者的效果是一样的。

## chunk
将大集合按指定大小拆分成小集合。

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);
$chunks = $collection->chunk(4);
$chunks->toArray();
// [[1, 2, 3, 4], [5, 6, 7]]
```

## collapse
将多个数组合并成一个集合。
```php
$collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);
// 注意这里返回了一个新的集合
$collapsed = $collection->collapse();
$collapsed->all();
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## combine
将一个集合的值作为「键」，再将另一个数组或者集合的值作为「值」合并成一个集合。

```php
$collection = collect(['name', 'age']);
$combined = $collection->combine(['boo', 25]);
$combined->all();
// ['name' => 'boo', 'age' => 25]
```

## contains
判断集合是否包含给定的项目。

基本用法：
```php
$collection = collect(['name' => 'boo', 'age' => 25]);
$collection->contains('boo'); // true
$collection->contains('yumi'); // false
```

也可以用 contains 方法匹配一对键/值，即判断给定的配对是否存在于集合中：
```php
$collection = collect([
    ['name' => 'boo', 'age' => 25],
    ['name' => 'yumi', 'age' => 23],
]);

$collection->contains("name", "mac");  // false
```

也可以传递一个回调到 contains 方法来执行自己的真实测试：
```php
$collection = collect([1, 2, 3, 4, 5]);

// $value: 1 $key: 0
$collection->contains(function ($value, $key) {
    return $value > 5;
}); // false
```
contains 方法在检查项目值时使用「宽松」比较，意味着具有整数值的字符串将被视为等于相同值的整数。 相反 containsStrict 方法则是使用「严格」比较进行过滤。

## containsStrict
使用「严格模式」判断集合是否包含给定的项目：

基本使用：
```php
$collection = collect([
    ['name' => 'boo', 'age' => 25],
    ['name' => 'yumi', 'age' => 23],
]);

$collection->containsStrict("age", "25");  // false
```
如上例所示，数组值存在，但是值类型不一致也返回false。

## count 
返回该集合内的项目总数。

```php
collect([1, 2, 3, 4])->count();  // 4
```

## diff
与给定的集合或者数组进行比较，基于值求差集。

将集合与其它集合或纯 PHP 数组进行值的比较，然后返回原集合中存在而给定集合中不存在的值：
```php
$collection = collect([1, 2, 3, 4, 5]);
$collection->diff([2, 4, 6, 8])->all();   // [1, 3, 5]
```

## diffAssoc
与给定的集合或者数组进行比较，基于键值对求差集。

返回原集合不存在于给定集合中的键值对：
```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6
]);

$diff->all(); // ['color' => 'orange', 'remain' => 6]
```

##diffKeys
与给定的集合或者数组进行比较，基于键求差集。

返回原集合中存在而给定的集合中不存在「键」所对应的键值对：
```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);

$diff->all(); // ['one' => 10, 'three' => 30, 'five' => 50]
```

## each
迭代集合中的内容并将其传递到回调函数中。

```php
$collection = $collection->each(function ($item, $key) {
    // 如果要中断对内容的迭代，那就从回调中返回 false
    if (/* some condition */) {
        return false;
    }
});
```

## except
返回集合中除了指定键以外的所有项目。

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);
$filtered = $collection->except(['price', 'discount']);
$filtered->all(); // ['product_id' => 1]
```
与之相反的方法是 `only()`。

## filter

使用给定的回调函数过滤集合的内容，只留下那些通过给定真实测试的内容。

```php
$collection = collect([1, 2, 3, 4]);
$filtered = $collection->filter(function ($value, $key) {
    // 当闭包返回true 时，保留一个条目
    return $value > 2;
});
$filtered->all(); // [3, 4]
```

如果没有提供回调函数，集合中所有返回false的元素都会被移除：
```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);
$collection->filter()->all(); // [1, 2, 3]
```

与之相反的方法是 `reject()`。

## first
返回集合中的第一个元素。

```
collect([1, 2, 3, 4])->first();  // 1
```

还可以传入一个闭包作为第二个函数：
```
collect([1, 2, 3, 4])->first(function ($value, $key) {
    // 当闭包返回true 时，保留一个条目
    return $value > 2;
}); // 3
```

如果需要返回最后一个元素可以使用`last()` 方法。

## firstMap
遍历集合并将其中的每个值传递到给定的回调。

可以通过回调修改每个值的内容再返回出来，从而形成一个新的被修改过内容的集合：
```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);
$flattened = $collection->flatMap(function ($values) {
    return array_map('strtoupper', $values);
});
$flattened->all();
// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

## flatten
将多维集合转为一维。

基本用法：
```php
$collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

$collection->flatten()->all();  // ['taylor', 'php', 'javascript'];
```

还可以选择性地传入「深度」参数：
```php
$collection = collect([
    'Apple' => [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
    ],
    'Samsung' => [
        ['name' => 'Galaxy S7', 'brand' => 'Samsung']
    ],
]);

$products = $collection->flatten(1);

$products->values()->all();

/*
[
    ['name' => 'iPhone 6S', 'brand' => 'Apple'],
    ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
]
*/
```
在这个例子里，调用 flatten 方法时不传入深度参数的话也会将嵌套数组转成一维的，然后返回 `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`，传入深度参数能限制设置返回数组的层数。

## flip
键值反转。

```php
$collection = collect(["name" => "boo", "age" => 25]);
$collection->flip()->all();  // ["boo" => "name", 25 => "age"]
```

## forget
通过给定的键来移除掉集合中对应的内容。

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
$collection->forget('name');
$collection->all(); 
// ['framework' => 'laravel']
```

与大多数集合的方法不同，`forget()` 不会返回修改过后的新集合；它会直接修改原来的集合。

## get
返回给定键的项目。

基本用法，如果该键不存在，则返回 null：
```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
$value = $collection->get('name'); // taylor
```

可以传递第二个参数作为默认值：
```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
$value = $collection->get('foo', 'boo'); // boo
```

甚至可以将回调函数当作默认值。如果指定的键不存在，就会返回回调的结果：
```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$collection->get('email', function () {
    return 'boo';
}); // boo
```

## groupBy
根据给定的键对集合内的项目进行分组。

## has
判断集合中是否存在给定的键。

```php
$collection = collect(["name" => "boo", "age" => 25]);
$collection->has("name");  // true
```

## implode
合并集合中的项目。


## intersect
从原集合中删除不在给定数组或集合中的任何值。


## intersectKey
删除原集合中不存在于给定数组或集合中的任何键。


## isEmpty
判断集合是否为空。


## isNotEmpty
判断集合是否不为空。


## keyBy
以给定的键作为集合的键。


## keys
返回集合的所有键。


## last
返回集合中通过给定真实测试的最后一个元素。


## map
遍历集合并将每一个值传入给定的回调。


## mapWithKeys
遍历集合并将每个值传入给定的回调。

## max
返回指定键的最大值。


## median
返回指定键的中间值。


## merge
将给定数组或集合合并到原集合。


## min
返回指定键的最小值。


## mode
返回指定键的众数值。

## nth 
创建由每隔n个元素组成一个新的集合。

## only 
返回集合中给定键的所有项目。


## partition 
配合list()方法区分回调函数满足和不满足的数据。


## pipe 
将集合传给给定的回调并返回结果。


## pluck 
获取集合中给定键对应的所有值。


## pop 
移除并返回集合中的最后一个项目。


## prepend 
将给定的值添加到集合的开头。


## pull 
把给定键对应的值从集合中移除并返回。


## push 
把给定值添加到集合的末尾。


## put 
在集合内设置给定的键值对。


## random 
从集合中返回一个随机项。

## reduce 
将每次迭代的结果传递给下一次迭代直到集合减少为单个值。


## reject 
使用指定的回调过滤集合。


## reverse 
倒转集合中项目的顺序。


## search
搜索给定的值并返回它的键。


## shift 
移除并返回集合的第一个项目。

## shuffle
随机排序集合中的项目。


## slice 
返回集合中给定值后面的部分。


## sort 
保留原数组的键，对集合进行排序。

## sortBy 	
以给定的键对集合进行排序。

## sortByDesc 
与sortBy一样，以相反的顺序来对集合进行排序。

## splice 
删除并返回从给定值后的内容，原集合也会受到影响。

## split 
将集合按给定的值拆分。


## sum 
返回集合内所有项目的总和。


## take
返回给定数量项目的新集合。


## tap
将集合传递给回调，在特定点「tap」集合。


## times 
通过回调在给定次数内创建一个新的集合。


## toArray
将集合转换成 PHP 数组。

## toJson 
将集合转换成 JSON 字符串。


## transform 
迭代集合并对集合内的每个项目调用给定的回调。


## union 
将给定的数组添加到集合中。


## unique 
返回集合中所有唯一的项目。


## uniqueStrict
使用严格模式返回集合中所有唯一的项目。


## values
返回键被重置为连续编号的新集合。


## when 
当传入的第一个参数为 true 的时，将执行给定的回调。


## where 
通过给定的键值过滤集合。


## whereStrict
使用严格模式通过给定的键值过滤集合。


## whereIn 
通过给定的键值数组来过滤集合。


## whereInStrict 
使用严格模式通过给定的键值数组来过滤集合。


## whereNotIn 
集合中不包含的给定键值对进行匹配。


## whereNotInStrict 
使用严格模式通过集合中不包含的给定键值对进行匹配。


## zip 
将给定数组的值与相应索引处的原集合的值合并在一起。


## 参考链接
* [Laravel 的集合 Collection](https://curder.gitbooks.io/laravel_study/content/collections/)