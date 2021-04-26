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

在非Laravel 项目中使用集合：
```shell
// 安装
composer require illuminate/support

// 使用
<?php
// 引入package
require __DIR__ . '/vendor/autoload.php';

$collection = collect([1, 2, 3]);
var_dump($collection);
```

记住，所有方法都可以使用链式编程的方式优雅的操纵数组。而且几乎所有的方法都会返回**新的** `Collection` 实例，

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

## chunkWhile
根据指定的回调值把集合分解成多个更小的集合：
```php
$collection = collect(str_split('AABBCCCD'));

$chunks = $collection->chunkWhile(function ($current, $key, $chunk) {
    return $current === $chunk->last();
});

$chunks->all();
// [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]
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

## collect
返回一个包含当前集合所含元素的新的 Collection 实例：
```php
$collection = collect([1, 2, 3]);
$collection->all(); 
// [1,2,3]
```

## concat
在集合的末端附加指定的数组或集合值：
```php
$collection = collect(['John Doe']);
$concatenated = $collection->concat(['Boo'])->concat(['name' => 'Yumi']);
$concatenated->all();
// ['John Doe', 'Boo', 'Yumi']
```

## contains
判断集合是否包含给定的项目。

基本用法：
```php
$collection = collect(['name' => 'boo', 'age' => 25]);
$collection->contains('boo');  // true
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

## countBy
统计集合中每个元素出现的次数。

基本用法：
```php
$collection = collect([1, 2, 2, 2, 3, 5, 5]);
$counted = $collection->countBy();
$counted->all();
// [1 => 1, 2 => 3, 3 => 1, 5=>2]
```

进阶用法，自定义规则，统计元素出现的次数：
```
$collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);
$counted = $collection->countBy(function ($email) {
    return substr(strrchr($email, "@"), 1);
});
$counted->all();
// ['gmail.com' => 2, 'yahoo.com' => 1]
```

## crossJoin
返回指定集合的可能的笛卡尔积。

```php
$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

## dd
备份文件系统和停止系统（dump and die）的缩写，打印集合元素并中断脚本执行。
```php
$collection = collect(['John Doe', 'Jane Doe']);
$collection->dd();
```
如果不想中断执行脚本，请使用`dump`方法替代。

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

## diffKeys
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

## duplicates
从集合中检索并返回重复的值。

基本用法：
```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);
$collection->duplicates();
// [2 => 'a', 4 => 'b']
```

如果集合包含数组或对象，则可以传递希望检查重复值的属性的键：
```php
$employees = collect([
    ['email' => 'abigail@example.com', 'position' => 'Developer'],
    ['email' => 'james@example.com', 'position' => 'Designer'],
    ['email' => 'victoria@example.com', 'position' => 'Developer'],
])

$employees->duplicates('position');
// [2 => 'Developer']
```
`duplicates` 方法在检查项目值时使用「宽松」比较，相反`duplicatesStrict` 方法则是使用「严格」比较进行过滤。

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

## eachSpread
同样是遍历集合，不过与each 的区别在于，对于多维数组，可以直接拿到元素。

```php
$collection = collect([['Boo', 25, "men"], ['Yumi', 23, "woman"]]);

$collection->eachSpread(function ($name, $age, $gender) {
    var_dump($name, $age, $gender);
    // Boo、25、men
    // Yumi、23、woman
});

$collection->each(function ($item, $key){
   // 同样可以在回调函数中，返回false ，终止循环
   var_dump($item, $key);
});
/*
    array(3) {
      [0]=>
      string(3) "Boo"
      [1]=>
      int(25)
      [2]=>
      string(3) "men"
    }
*/
```

## every
检查集合中的每一个元素是否通过指定条件：

```
collect([1, 2, 3, 4])->every(function ($value, $key) {
    return $value > 2;
});

// false
```
注意：如果集合为空， every 将返回 true。

## except
返回集合中除了指定键以外的所有项目。

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);
$filtered = $collection->except(['price', 'discount']);
$filtered->all(); // ['product_id' => 1]
```
与之相反的方法是 `only()`。

## filter
使用给定的回调函数过滤集合的内容，只留下那些通过的元素。

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

基本用法：
```php
collect([1, 2, 3, 4])->first();  // 1
```

同样可以传入回调函数，进行条件限制：
```php
collect([1, 2, 3, 4])->first(function ($value, $key) {
    // 当闭包返回true 时，保留一个条目
    return $value > 2;
}); // 3
```

如果需要返回最后一个元素可以使用`last()` 方法。

## firstWhere
返回集合中含有指定键 / 值对的第一个元素：

```php
$collection = collect([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);

// 返回name = Linda 的第一个元素
$collection->firstWhere('name', 'Linda');
// ['name' => 'Linda', 'age' => 14]
```

还可以在firstWhere 中使用算术运算符：
```php
$collection->firstWhere('age', '>=', 18);
// ['name' => 'Diego', 'age' => 23]
```

和 where 方法一样，你可以将一个参数传递给 firstWhere 方法。在这种情况下， firstWhere 方法将返回指定键的值为「真」的第一个集合项：
```php
$collection->firstWhere('age');
// ['name' => 'Linda', 'age' => 14]
```

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

基本用法：
```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->all();

/*
    [
        'account-x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'account-x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

同样可以传入一个回调函数来代替字符串的『键』，根据该回调函数的返回值来进行分组：
```php
$grouped = $collection->groupBy(function ($item, $key) {
    return substr($item['account_id'], -3);
});

$grouped->all();

/*
    [
        'x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

甚至可以传入一个数组进行多层分组：
```php
$data = new Collection([
    10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
    20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
    30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
    40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
]);

$result = $data->groupBy([
    'skill',
    function ($item) {
        return $item['roles'];
    },
], $preserveKeys = true);

/*
[
    1 => [
        'Role_1' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_2' => [
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_3' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        ],
    ],
    2 => [
        'Role_1' => [
            30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        ],
        'Role_2' => [
            40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
        ],
    ],
];
*/
```

## has
判断集合中是否存在给定的键。

```php
$collection = collect(["name" => "boo", "age" => 25]);
$collection->has("name");  // true
```

## implode
合并集合中的项目。

implode 方法用于合并集合项。其参数取决于集合项的类型。如果集合包含数组或对象，你应该传递你希望合并的属性的键，以及你希望放在值之间用来「拼接」的字符串：
```php
$collection = collect([
    ['account_id' => 1, 'product' => 'Desk'],
    ['account_id' => 2, 'product' => 'Chair'],
]);

$collection->implode('product', ', ');
// Desk, Chair
```

如果集合中包含简单的字符串或数值，只需要传入「拼接」用的字符串作为该方法的唯一参数即可：
```php
collect([1, 2, 3, 4, 5])->implode('-');

// '1-2-3-4-5'
```

## intersect
从原集合中移除不在给定数组或集合中的『任何值』，返回新的集合将保留原集合的键。

```php
$collection = collect(['Desk', 'Sofa', 'Chair']);
$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);
$intersect->all();
// [0 => 'Desk', 2 => 'Chair']
```

## intersectKey
删除原集合中不存在于给定数组或集合中的『任何键』，返回新的集合将保留原集合的键。

```php
$collection = collect([
    'serial' => 'UX301', 'type' => 'screen', 'year' => 2009,
]);

$intersect = $collection->intersectByKeys([
    'reference' => 'UX404', 'type' => 'tab', 'year' => 2011,
]);
$intersect->all();
// ['type' => 'screen', 'year' => 2009]
```

## isEmpty
判断集合是否为空。

```php
collect([])->isEmpty(); // true
```

## isNotEmpty
判断集合是否不为空。

```php
collect([])->isEmpty(); // false
```

## join
将集合中的值用字符串连接。

```php
collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
collect(['a'])->join(', ', ' and '); // 'a'
collect([])->join(', ', ' and '); // ''
```

## keyBy
以给定的键作为集合的键。

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

还可以在这个方法传递一个回调函数。该回调函数返回的值会作为该集合的键：
```php
$keyed = $collection->keyBy(function ($item) {
    return strtoupper($item['product_id']);
});

$keyed->all();

/*
    [
        'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

## keys
返回集合的所有键。

```php
$collection = collect(["name" => "boo", "age" => 25]);
$collection->keys()->all();  // ["name", "age"]
```

## last
返回集合中通过给定真实测试的最后一个元素，与`first` 方法正好相反。

```php
collect([1, 2, 3, 4])->last(function ($value, $key) {
    return $value < 3;
});
// 2
```

## map
遍历集合并将每一个值传入给定的回调，返回新的集合。

```php
$collection = collect([1, 2, 3, 4, 5]);

$multiplied = $collection->map(function ($item, $key) {
    return $item * 2;
});

$multiplied->all();
// [2, 4, 6, 8, 10]
```
与其他大多数集合方法一样， map 会返回一个新的集合实例；它不会修改原集合。如果你想修改原集合，请使用 `transform` 方法。

## mapToGroups
通过指定回调函数对集合进行分组，

```php
$collection = collect([
    [
        'name' => 'John Doe',
        'department' => 'Sales',
    ],
    [
        'name' => 'Jane Doe',
        'department' => 'Sales',
    ],
    [
        'name' => 'Johnny Doe',
        'department' => 'Marketing',
    ]
]);

$grouped = $collection->mapToGroups(function ($item, $key) {
    return [$item['department'] => $item['name']];
});

$grouped->all();

/*
    [
        'Sales' => ['John Doe', 'Jane Doe'],
        'Marketing' => ['Johnny Doe'],
    ]
*/

$grouped->get('Sales')->all();
// ['John Doe', 'Jane Doe']
```

## mapWithKeys
遍历集合并将每个值传入给定的回调。

## max
返回指定键的最大值。

```php
$max = collect([['foo' => 10], ['foo' => 20]])->max('foo');   // 20
$max = collect([1, 2, 3, 4, 5])->max();   // 5
```

## median
返回指定键的中间值。

```php
$median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');   // 15
$median = collect([1, 1, 2, 4])->median();  // 1.5
```

## merge
将给定数组或集合合并到原集合。

如果给定的集合项的字符串键与原集合中的字符串键相匹配，则指定集合项的值将覆盖原集合的值：
```php
$collection = collect(['product_id' => 1, 'price' => 100]);
$merged = $collection->merge(['price' => 200, 'discount' => false]);
$merged->all();
// ['product_id' => 1, 'price' => 200, 'discount' => false]
```

如果给定的集合项为数字，则这些值将会追加在集合的最后：
```
$collection = collect(['Desk', 'Chair']);
$merged = $collection->merge(['Bookcase', 'Door']);
$merged->all();
// ['Desk', 'Chair', 'Bookcase', 'Door']
```

## min
返回指定键的最小值。

```php
$min = collect([1, 2, 3, 4, 5])->min();     // 1
```

## mode
返回指定键的众数值。

```php
collect([1, 1, 2, 4])->mode();    // [1]
```

## nth 
每隔n个元素取一个元素组成一个新的集合。

```php
$collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);
$collection->nth(4);    // ['a', 'e']

// 第二个参数可以作为偏移位置 
$collection->nth(4, 1); // ['b', 'f']
```

## only 
返回集合中给定键的所有项目。

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);
$filtered = $collection->only(['product_id', 'name']);
$filtered->all();     // ['product_id' => 1, 'name' => 'Desk']
```

## partition 
配合list()方法区分回调函数满足和不满足的数据。

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

list($underThree, $equalOrAboveThree) = $collection->partition(function ($i) {
    return $i < 3;
});
$underThree->all();     // [1, 2]
$equalOrAboveThree->all();    // [3, 4, 5, 6]
```

## pipe 
将集合传给给定的回调并返回结果。
```
$collection = collect([1, 2, 3]);
$piped = $collection->pipe(function ($collection) {
    return $collection->sum();
}); // 6
```

## pluck 
获取集合中给定键对应的所有值。

基本用法：
```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk', "id" => 1],
    ['product_id' => 'prod-200', 'name' => 'Chair', "id" => 2],
]);

$plucked = $collection->pluck('name');
$plucked->all();    // ['Desk', 'Chair']
```

还可以传入第二个参数作为键值：
```
$plucked = $collection->pluck('name', "id");
$plucked->all();    // [1 => 'Desk', 2 => 'Chair']
```

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
* [Laravel 集合——Laravel8.x 中文文档](https://learnku.com/docs/laravel/8.x/collections/9390)